# T05 인수인계 문서 — 지역 선택 드롭다운 (AI B)

이 저장소는 **index.html 한 파일**(HTML+CSS+JS 전부 포함, 더블클릭으로 바로 실행)로 구성됩니다.
`contract/`, `fixtures/`는 과제에서 원본으로 받은 T04 계약 문서·fixture로, 참고용 원본이며
이번 T05 작업에서 건드리지 않았습니다.

## 작업 기록

| 단계 | 시각(UTC) / 라운드트립 | 소스 버전(commit) | 비고 |
|---|---|---|---|
| AI A 시작 | 2026-09-10T01:28:22Z / 왕복 1 | `0b4161f...` | T04 완성본 baseline |
| AI A 종료·인수인계 | 왕복 3 | (AI A 커밋) | 파일 구조 정리 |
| **AI B 시작** | **(미기입 — 시작 시 채움)** | **(미기입)** | 이 문서가 AI B 산출물 |
| **AI B 종료** | **(미기입)** | **(미기입)** | |

**공통 상한:** 시간 10분(사고·생성만) / 왕복 3회.

**최초 요청:** "T04 정보판에 지역 선택 드롭다운(서울/대전/부산)을 추가하라.
드롭다운으로 지역을 바꾸면 그 지역 좌표로 라이브 조회를 하고,
지역별로 별도 signal_id로 기록을 쌓아 보여준다."

---

## 1. 목표

T04 "오늘의 진짜 정보판"에 **지역 선택 드롭다운**을 추가한다. 서울/대전/부산 중 하나를 고르면
그 지역 좌표로 Open-Meteo를 조회하고, 지역별로 서로 다른 `signal_id`로 기록을 구분해 쌓되
화면은 선택된 지역 것만 보여준다 (다른 지역 기록은 유지).

## 2. 현재 상태

**핵심 설계 — 검사 10개 대응:**

| 검사 | 대응 |
|---|---|
| 01 기본값 서울 | `currentRegionId` 초기값 `'seoul'` |
| 02/03 값·라벨 갱신 | `onRegionChange` → `renderLive()` 즉시 반영 후 `refreshLive()` |
| 04 다른 지역 기록 보존 | **지역별 독립 T04 state** (`regionStates[regionId]`) |
| 05 같은 지역 재선택 중복 없음 | T04Core의 `applySuccessfulReading` (signal_id+record_date 매칭) 그대로 |
| 06 표 필터링 | 각 지역 state의 `daily_readings`가 원래 그 지역 행만 담음 |
| 07 오류 격리 | 상태가 지역별로 분리 → 한 지역 오류가 다른 지역에 안 번짐 |
| 08 새로고침 유지 | `current_region_id`를 localStorage에 저장, 초기 로드에서 복원 |
| 09 저장 파일에 다지역 | 저장 JSON이 `{current_region_id, regions:{seoul,daejeon,busan}}` |
| 10 경쟁 조건 | `requestSeq` 증가 가드: 최신 요청이 아니면 결과 폐기 |

**index.html 내부 구조** (`<style>` 1개 + `<script>` 1개, 5개 구획):
1. **CORE** — T04 계약 포팅 (수정 금지 구간)
2. **LIVE-SOURCE** — `REGIONS` 배열(3개 지역), `fetchLiveReading(regionId)`
3. **STORAGE** — localStorage(`aleph_t05_state_v1`) + 파일 저장/불러오기
4. **FIXTURES-DATA** — 9개 결정론 fixture 내장
5. **MAIN** — 렌더, 지역 드롭다운, `requestSeq` 경쟁조건 가드, 진단 패널

**지역 정의:**
| id | 라벨 | 위도/경도 | signal_id |
|---|---|---|---|
| seoul | 서울, 대한민국 | 37.5665, 126.9780 | kr-seoul-temp-c |
| daejeon | 대전, 대한민국 | 36.3504, 127.3845 | kr-daejeon-temp-c |
| busan | 부산, 대한민국 | 35.1796, 129.0756 | kr-busan-temp-c |

## 3. 실행 명령

```bash
open index.html     # 또는 더블클릭 (file:// 그대로 동작)

python3 -m http.server 8000   # 여러 사람과 공유할 때
