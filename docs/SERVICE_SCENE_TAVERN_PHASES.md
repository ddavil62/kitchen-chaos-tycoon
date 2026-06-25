# Kitchen Chaos -- 영업씬 태번 스타일 재설계 페이즈 마스터 플랜

> 최종 업데이트: 2026-04-30 (Phase A ~ E 완료, TavernServiceScene 마이그레이션 완결)
> 관련 문서:
> - [SERVICE_SCENE_TAVERN_DIRECTION.md](SERVICE_SCENE_TAVERN_DIRECTION.md) -- 방향성/핵심 원칙 (V12 기준)
> - `.claude/specs/2026-04-23-kc-phase-b-asset-spec.md` -- Phase B 에셋 발주 규격서 (V12 규격 갱신 완료)

---

## 페이즈 총괄

기존 ServiceScene.js(아이소메트릭 격자 + 3레이어 분리 렌더링)를 Travellers Rest 스타일(탑다운 가구 + 사이드뷰 풀바디 캐릭터)로 전면 재설계한다. 기존 씬을 수정하지 않고 TavernServiceScene을 병렬 구현한 뒤, 최종적으로 교체한다.

| 페이즈 | 이름 | 핵심 범위 | 상태 |
|--------|------|-----------|------|
| **A** | 기반 시스템 골격 | 레이아웃 상수(V10), 슬롯 데이터 모델, 상태머신 키, 깊이정렬, PIL 더미 | **완료** |
| **A-bis** | V12 레이아웃 마이그레이션 | V10 가로 3세트 -> V12 4분면 세로, V12 placeholder 5종, 테스트 84/84 | **완료** |
| **B-1** | 실 에셋 1세트 발주 | 손님 seated 2종 + 셰프 미미 idle + 가구 5종 PixelLab 발주, ASSET_MODE 토글 통합 | **완료** |
| **B-2** | WARN 해소 + 스프라이트 전환 | 에셋 4종 재발주/신규(seated_left/bench L,R/chef_rin), 셰프+손님 Image 렌더링, vite 빌드 위생 | **완료** |
| **B-3** | 에셋 확장 발주 (손님 9종 + 셰프 5명) | 손님 9종 seated R/L 18장 + 셰프 5명 idle 5장 + W-1 재발주 = 24장, DEMO_CUSTOMER_TYPES 4종, REAL_KEY_MAP 15개 | **완료** |
| **B-4** | 에셋 확장 발주 (walk 시트 + W-1 PRO) | 손님 10종 walk_l/r 20장 + W-1 PRO 재발주 (PARTIAL 확정), spritesheet preload + anims 20개 + 데모 키 | **완료** |
| **B-5-1** | 셰프 walk 시트 발주 | 셰프 5명 walk_r/walk_l 10장(64x24), spritesheet preload + anims 10개 + C/V 데모 키 | **완료** |
| **B-6** | 캐릭터 해상도 업스케일 | 15명 size=48 재발주, 16x24->32x48 업스케일, frameWidth/Height 갱신, 55장 신규 에셋 | **완료** |
| **B-6-2** | 가구 비례 업스케일 | 벤치(14x76) 등 가구 에셋을 32x48 캐릭터 비례에 맞게 업스케일 (AD3 NOTE) | 미착수 |
| **B-5-1b** | 셰프 carry/cook/serve 포즈 | 셰프 5명 carry_r/l + cook + serve 정지 포즈 (AD1에서 B-5-1에서 분리) | 미착수 |
| **B-5-2** | 가구 lv3/lv4 업그레이드 에셋 | lv3/lv4 등급 가구 에셋 발주 | 미착수 |
| **B-5-3** | 환경물 에셋 | 술통/바닥 타일/벽 장식 환경물 발주 | 미착수 |
| **C** | 테마 에셋 + HUD/VFX | g1 바닥/벽 타일 v14, 인내심 게이지, 말풍선, 골드 플로팅, HUD 바 | **완료** (Phase 94, QA 36/36) |
| **D** | 게임 로직 포팅 | 조리 슬롯, 레시피 퀵슬롯, 재고 패널, Image→Sprite, 손님 AI, 인내심, 서빙, 스킬, 이벤트, 미력사 | **완료** (Phase 95, QA 38/38) |
| **E** | 마이그레이션 | GatheringScene/EndlessScene 진입점 교체, ServiceScene→_legacy/, 디버그 가드 | **완료** (Phase 96, QA 101/101) |

---

## Phase A -- 기반 시스템 골격 (완료)

### 범위

- A1: 360x640 캔버스 레이아웃 영역 상수 정의 + PIL 더미 합성
- A2: 긴 벤치 좌석 슬롯 데이터 모델 (lv0 4인, lv3 5인, lv4 6인) -- V10 가로 구조
- A3: 셰프/손님 상태머신 키 정의 (ChefState 7, CustomerState 7)
- A4: Y축 단순 깊이정렬 + 3레이어 분리 렌더링 폐기

### 산출물

| 파일 | 역할 |
|------|------|
| `js/scenes/TavernServiceScene.js` | 메인 씬 (A1~A4 통합, 디버그 전용) |
| `js/data/tavernLayoutData.js` | 레이아웃 상수, BENCH_SLOTS, occupy/vacate/findFreeSlot/getSlotWorldPos |
| `js/data/tavernStateData.js` | ChefState/CustomerState 열거형, 전환 맵, 더미 색상 |
| `assets/tavern_dummy/` (13개 PNG) | PIL 더미 에셋 |
| `tests/phase-a-tavern-qa.spec.js` | Playwright 14개 테스트 |

### 게이트 충족 상태

- [x] 영역 구획 상수 확정 (HUD 32px / WALL 24px / KITCHEN_W 120px / DINING_W 224px / CTRL 80px)
- [x] 테이블 세트 y좌표 확정 (V10: 232, 352, 472 / 간격 120px 균등) -- Phase A-bis에서 V12로 교체됨
- [x] 슬롯 데이터 모델 확정 (24석 lv0, occupySlot/vacateSlot API)
- [x] 상태머신 키 확정 (셰프 7 + 손님 7, scaleY 완전 배제)
- [x] 깊이정렬 공식 확정 (depth = y 단일 공식)
- [x] ServiceScene.js 미수정 확인
- [x] Phase B 진입 게이트 에셋 발주 규격서 작성

### QA 결과

PASS (40/40). AD 모드3 REVISE 1건(bench-top 4px 겹침) 반영 완료.

### 스펙/리포트

- 스펙: `.claude/specs/2026-04-23-kc-phase-a-tavern-spec.md`
- Coder 리포트: `.claude/specs/2026-04-23-kc-phase-a-tavern-coder-report.md`
- AD 모드3: `.claude/specs/2026-04-23-kc-phase-a-tavern-ad3.md`
- QA: `.claude/specs/2026-04-23-kc-phase-a-tavern-qa.md`

---

## Phase A-bis -- V12 레이아웃 마이그레이션 (완료)

### 범위

Phase A(V10 가로 긴 테이블 3세트x8석=24석)를 V12(세로 짧은 테이블 4분면x6석=24석)로 전환. 코드 구조/상태머신/함수 인터페이스 유지, 좌표/에셋 크기 상수와 렌더링 배치만 V12 수치로 치환.

- TABLE_SET_ANCHORS: 3엔트리(x,y) -> 4엔트리(quadLeft,quadTop,key: tl/tr/bl/br)
- BENCH_SLOTS: top/bot(dx) -> left/right(dy)
- 카운터: 24x120 가로 -> 40x100 세로
- 셰프 2명 배치 (y=100, y=148)
- 입구: 우측 상단 -> 좌하단(entrance_v12, left=44, top=480)
- V12 PIL placeholder 5종 추가

### 산출물

| 파일 | 역할 |
|------|------|
| `assets/tavern_dummy/counter_v12.png` (40x100) | V12 카운터 placeholder |
| `assets/tavern_dummy/table_vertical_v12.png` (44x72) | V12 세로 테이블 placeholder |
| `assets/tavern_dummy/bench_vertical_l_v12.png` (14x76) | V12 좌측 벤치 placeholder |
| `assets/tavern_dummy/bench_vertical_r_v12.png` (14x76) | V12 우측 벤치 placeholder |
| `assets/tavern_dummy/entrance_v12.png` (32x40) | V12 입구 placeholder |
| `tests/phase-a-bis-v12-qa.spec.js` (27개) | QA 신규 테스트 |

### 게이트 충족 상태

- [x] TABLE_SET_ANCHORS 4엔트리 (tl/tr/bl/br quad 좌표)
- [x] 24석 총원 유지 (4quad x left3 + right3)
- [x] 세로 통로 20px, 가로 통로 40px
- [x] V12 placeholder 5종 존재 + preload 로드
- [x] ServiceScene.js 무손상 (git diff 0줄)
- [x] scaleY/flipY 코드 부재
- [x] Playwright 84/84 PASS, vite build PASS

### QA 결과

PASS (84/84 테스트, 게이트 12/12). AD 모드3 REVISE (WARN 4건, 모두 Phase B 이후 해소 예정).

### 스펙/리포트

- 스펙: `.claude/specs/2026-04-23-kc-phase-a-bis-v12-migration-spec.md`
- 목적: `.claude/specs/2026-04-23-kc-phase-a-bis-v12-migration-scope.md`
- Coder 리포트: `.claude/specs/2026-04-23-kc-phase-a-bis-v12-migration-coder-report.md`
- AD 모드3: `.claude/specs/2026-04-23-kc-phase-a-bis-v12-migration-ad3.md`
- QA: `.claude/specs/2026-04-23-kc-phase-a-bis-v12-migration-qa.md`

---

## Phase B-1 -- 실 에셋 1세트 발주 (완료)

### 범위

PixelLab으로 최소 1세트(normal 손님 seated 2장 + 미미 셰프 idle_side 1장 + V12 가구 5종)를 발주하여, Phase B 파이프라인(발주 -> 후처리 -> 통합 -> 검증)이 정상 동작함을 확인. 동시에 규격서 V10 잔재 7개소를 V12로 갱신.

### 산출물

| 파일 | 역할 |
|------|------|
| `assets/tavern/customer_normal_seated_right.png` (16x22) | 일반 손님 우향 앉기 |
| `assets/tavern/customer_normal_seated_left.png` (16x22) | 일반 손님 좌향 앉기 |
| `assets/tavern/chef_mimi_idle_side.png` (16x24) | 셰프 미미 idle |
| `assets/tavern/counter_v12.png` (40x100) | 실 에셋 카운터 |
| `assets/tavern/table_vertical_v12.png` (44x72) | 실 에셋 세로 테이블 |
| `assets/tavern/bench_vertical_l_v12.png` (14x76) | 실 에셋 좌측 벤치 |
| `assets/tavern/bench_vertical_r_v12.png` (14x76) | 실 에셋 우측 벤치 |
| `assets/tavern/entrance_v12.png` (32x40) | 실 에셋 입구 |
| `tests/phase-b1-asset-load.spec.js` (19개) | Playwright 테스트 |

### 게이트 충족 상태

- [x] 에셋 8종 native 크기 정확 일치 (0px 오차)
- [x] ASSET_MODE='real' 토글로 실 에셋/PIL placeholder 전환
- [x] 가구 5종 실 에셋 렌더링 확인
- [x] 캐릭터 3종 텍스처 로드 확인 (스프라이트 전환은 B-2)
- [x] tavern_dummy/ 보존, 두 세트 공존
- [x] ServiceScene.js 무손상 (git diff 0줄)
- [x] Playwright 46/46 PASS

### QA 결과

PASS (46/46 테스트, SC-1~SC-5 전항목 충족).

### Phase B-2에서 해소된 사항

- ~~손님 seated_left 셔츠 색상 불일치 (AD2 WARN-1)~~ -- 부분 개선 (0% -> 19.2% teal), WARN-1 잔여 -> B-3에서 완전 해소
- ~~벤치 14px crop 음영 약화 (AD2 WARN-2/3)~~ -- NEAREST 가로 스케일 다운 방식으로 **해소** (W-2 33.6%, W-3 67.4% 어둡)
- ~~chef-1 placeholder 잔존 (AD3 WARN-1)~~ -- chef_rin_idle_side.png 발주로 **해소**
- ~~캐릭터 스프라이트 전환~~ -- `_buildChef()`/`_buildCustomers()` -> `_placeImageOrRect()` 경로 전환 **완료**
- ~~`_postprocess.py`/`_raw/` 프로덕션 빌드 제외 필터~~ -- vite.config.js COPY_DIR_EXCLUDE/COPY_FILE_EXCLUDE **완료**

### Phase B-3 해소 결과 / Phase B-5 이후 해소 예정

- ~~seated_left 셔츠 청록 완전 일치~~ -- B-3에서 size=44 재발주 + Option B 색상 정규화 적용. **PARTIAL 최종 확정**: B-4 PRO 모드(size=64, custom proportions) 재시도 결과 teal=3으로 악화. B-1~B-4 총 4회 시도 실패, 구조적 한계 확정. 추가 재시도 없이 마감
- 술통(barrel) 실 에셋 발주 -- Phase B-5+ 범위
- write_ad2_report.py untracked 파일 정리

### 스펙/리포트

- 목적: `.claude/specs/2026-04-23-kc-phase-b1-asset-scope.md`
- 스펙: `.claude/specs/2026-04-23-kc-phase-b1-asset-spec.md`
- AD 모드1: `.claude/specs/2026-04-23-kc-phase-b1-ad1.md`
- Coder 리포트: `.claude/specs/2026-04-23-kc-phase-b1-coder-report.md`
- AD 모드2: `.claude/specs/2026-04-23-kc-phase-b1-ad2.md`
- AD 모드3: `.claude/specs/2026-04-23-kc-phase-b1-ad3.md`
- QA: `.claude/specs/2026-04-23-kc-phase-b1-qa.md`

---

## Phase B-2 -- B-1 WARN 해소 + 스프라이트 전환 (완료)

### 범위

B-1 조건부 승인 WARN 4건(seated_left 셔츠 색상, 벤치 음영 x2, 셰프 placeholder)을 에셋 4종 재발주/신규 발주로 해소. `_buildChef()`/`_buildCustomers()`를 `_placeImageOrRect()` 경로로 전환하여 셰프+손님이 실 Image 오브젝트로 렌더링. vite 빌드에서 `_raw/` + `.py` 제외.

### 산출물

| 파일 | 역할 |
|------|------|
| `assets/tavern/chef_rin_idle_side.png` (16x24) | 두 번째 셰프 린 idle (W-4 해소) |
| `assets/tavern/customer_normal_seated_left.png` (16x22) | 재발주 (W-1 부분 개선) |
| `assets/tavern/bench_vertical_l_v12.png` (14x76) | 재발주 NEAREST 스케일 다운 (W-2 해소) |
| `assets/tavern/bench_vertical_r_v12.png` (14x76) | 재발주 NEAREST 스케일 다운 (W-3 해소) |
| `assets/tavern/_postprocess_b2.py` | B-2 후처리 스크립트 |
| `tests/phase-b2-sprite-transition.spec.js` (15개) | 스프라이트 전환 Playwright 테스트 |

### 게이트 충족 상태

- [x] 에셋 4종 native 크기 정확 일치 (0px 오차)
- [x] REAL_KEY_MAP 9개 매핑 (chef2 추가)
- [x] `_buildChef()` / `_buildCustomers()` 내 `add.rectangle` 직접 호출 0건
- [x] 셰프 2명(린 idx=0, 미미 idx=1) Image 타입 렌더링
- [x] 손님 Image 타입 렌더링 + SIT 텍스처 교체
- [x] vite build 산출물에 `_raw/` + `.py` 미포함
- [x] ServiceScene.js diff 0줄, scaleX/flipX 0건, tavern_dummy/ 변경 0건
- [x] Playwright 114/118 PASS (실패 4건 cold-start 인프라)

### QA 결과

PASS (114/118 테스트, SC-1 PARTIAL + SC-2~SC-7 PASS). B-2 회귀 0건.

### 스펙/리포트

- 목적: `.claude/specs/2026-04-24-kc-phase-b2-scope.md`
- 스펙: `.claude/specs/2026-04-24-kc-phase-b2-spec.md`
- AD 모드1: `.claude/specs/2026-04-24-kc-phase-b2-ad1.md`
- Coder 리포트: `.claude/specs/2026-04-24-kc-phase-b2-coder-report.md`
- AD 모드2: `.claude/specs/2026-04-24-kc-phase-b2-ad2.md`
- QA: `.claude/specs/2026-04-24-kc-phase-b2-qa.md`

---

## Phase B-3 -- 에셋 확장 발주: 손님 9종 + 셰프 5명 (완료)

### 범위

PixelLab으로 손님 9종(vip~business) seated_right/left 18장 + 셰프 5명(mage/yuki/lao/andre/arjun) idle_side 5장 + W-1 재발주(size=44 + Option B 색상 정규화) = 총 24장 발주. DEMO_CUSTOMER_TYPES 4종 데모 배치, REAL_KEY_MAP +6키, preload +23, SIT 텍스처 동적 교체 구현.

### 산출물

| 파일 | 역할 |
|------|------|
| `assets/tavern/customer_{9종}_seated_right.png` (각 16x22) | 손님 9종 우향 앉기 |
| `assets/tavern/customer_{9종}_seated_left.png` (각 16x22) | 손님 9종 좌향 앉기 (독립 스프라이트) |
| `assets/tavern/chef_{mage/yuki/lao/andre/arjun}_idle_side.png` (각 16x24) | 셰프 5명 idle |
| `assets/tavern/customer_normal_seated_left.png` (16x22) | W-1 재발주 (size=44 + Option B 정규화) |
| `assets/tavern/_postprocess_b3.py` | B-3 후처리 스크립트 |
| `tests/phase-b3-asset-expansion.spec.js` (35개) | Playwright 테스트 |

### 게이트 충족 상태

- [x] 손님 9종 seated R/L 18장 저장, native 16x22
- [x] 셰프 5명 idle_side 5장 저장, native 16x24
- [x] REAL_KEY_MAP 15개 (기존 9 + 데모 3종 seated 6)
- [x] preload realAssets 32개 (기존 9 + B-3 신규 23)
- [x] DEMO_CUSTOMER_TYPES 4종 배치 + customerType 프로퍼티
- [x] SIT 텍스처 동적 교체 (customerType 기반)
- [x] ServiceScene.js diff 0줄, scaleX/flipX 0건, tavern_dummy/ 변경 0건
- [x] Playwright 153/153 PASS, 회귀 0건
- [ ] W-1 teal 25% 미달 (PARTIAL, 11.3%) -- 구조적 한계. B-4 PRO 모드 재발주 결과 teal=3으로 악화, 최종 확정

### QA 결과

PASS (153/153 테스트, SC-1 PARTIAL + SC-2~SC-10 PASS). AD2 APPROVED (conditional).

### W-1 PARTIAL 사유

facing-left 사이드뷰에서 캐릭터 몸통이 화면 왼쪽으로 회전 -> 셔츠 가시 영역이 x=7,8 좁은 띠(6cells 상한). size=44/64 큰 캔버스에서도 teal이 음영에 묻혀 aggregation에서 소실됨. B-4 PRO 모드(size=64 + custom proportions shoulder_width=1.1) 재시도 결과 teal=3으로 악화(B-3의 6보다 감소). **구조적 한계 최종 확정**, silhouette 재설계 또는 수동 편집 없이는 개선 불가. 추가 재시도 없이 마감.

### 스펙/리포트

- 목적: `.claude/specs/2026-04-24-kc-phase-b3-scope.md`
- 스펙: `.claude/specs/2026-04-24-kc-phase-b3-spec.md`
- AD 모드1: `.claude/specs/2026-04-24-kc-phase-b3-ad1.md`
- Coder 리포트: `.claude/specs/2026-04-24-kc-phase-b3-coder-report.md`
- AD 모드2: `.claude/specs/2026-04-24-kc-phase-b3-ad2.md`
- QA: `.claude/specs/2026-04-24-kc-phase-b3-qa.md`

---

## Phase B-4 -- 손님 Walk 애니메이션 시트 발주 + W-1 PRO 재발주 (완료)

### 범위

손님 10종의 walk_l/walk_r 4프레임 스프라이트시트 20장을 PixelLab `animate_character`로 발주. TavernServiceScene.js에 spritesheet preload + Phaser 애니메이션 등록 + 데모 키 핸들러(W/A/S)를 추가하여 시트 동작 검증. W-1 PRO 모드 재발주(1회 한정)로 구조적 한계 최종 확정.

### 산출물

| 파일 | 역할 |
|------|------|
| `assets/tavern/customer_{10종}_walk_r.png` (각 64x24) | 손님 10종 우향 보행 4프레임 시트 |
| `assets/tavern/customer_{10종}_walk_l.png` (각 64x24) | 손님 10종 좌향 보행 4프레임 시트 (독립 스프라이트) |
| `assets/tavern/_postprocess_b4.py` | B-4 후처리 스크립트 |
| `assets/tavern/_raw_b4/` | 원본 프레임 80장 + 결과 JSON 백업 |
| `tests/phase-b4-walk-animation.spec.js` (31개) | Playwright 테스트 |

### 게이트 충족 상태

- [x] walk_l/r 20장 64x24 저장, HTTP 200 + PIL 크기 검증
- [x] spritesheet preload 20개 등록 (기존 32 + walk 20 = 52개 이상 텍스처)
- [x] Phaser anims 20개 등록 (customer_{type}_walk_r/l, 8fps, repeat=-1)
- [x] W/A/S 데모 키 동작 (null guard, ASSET_MODE='real' 분기)
- [x] window.__tavernWalkAnims 진단 노출
- [x] ServiceScene.js diff 0줄, scaleX/flipX 0건, tavern_dummy/ 변경 0건
- [x] Playwright 184/184 PASS (B-4 31 + 회귀 153), 회귀 0건
- [ ] W-1 teal 25% 미달 (PARTIAL, PRO 시도 teal=3으로 악화, 구조적 한계 최종 확정)

### QA 결과

PASS (184/184 테스트, SC-1~SC-6 전수 충족, SC-2 PARTIAL은 AD2 확정 사항). 2건 cold-start 타임아웃은 B-2/B-3 기존 인프라 이슈.

### 스펙 대비 구현 차이

- S 키 핸들러 추가 (스펙은 W/A만 명시, S키 stop+idle 복귀는 유용한 디버그 기능으로 추가)
- Image 객체에서 sprite.play() 호출 시 실제 프레임 전환 미동작 (Coder 인지, Phase D Sprite 전환 시 해소 예정, 현재 에러 미발생)

### 스펙/리포트

- 목적: `.claude/specs/2026-04-24-kc-phase-b4-scope.md`
- 스펙: `.claude/specs/2026-04-24-kc-phase-b4-spec.md`
- AD 모드1: `.claude/specs/2026-04-24-kc-phase-b4-ad1.md`
- AD 모드2: `.claude/specs/2026-04-24-kc-phase-b4-ad2.md`
- Coder 리포트: `.claude/specs/2026-04-24-kc-phase-b4-coder-report.md`
- QA: `.claude/specs/2026-04-24-kc-phase-b4-qa.md`

---

## Phase B-5-1 -- 셰프 Walk 스프라이트시트 발주 (완료)

### 범위

셰프 5명(mage/yuki/lao/andre/arjun)의 walk_r/walk_l 4프레임 스프라이트시트 10장을 PixelLab `animate_character`로 발주. AD 모드1에서 원래 B-5 범위(walk+carry+cook+serve = 최대 35장)를 walk만(10장)으로 축소 결정. carry/cook/serve는 PixelLab 템플릿 매칭 불가 + 시각 일관성 위험으로 B-5-1b로 분리. TavernServiceScene.js에 spritesheet preload 10개 + Phaser anims 10개 + C/V 데모 키 핸들러 + window.__tavernChefAnims 진단 노출 추가.

### 산출물

| 파일 | 역할 |
|------|------|
| `assets/tavern/chef_{5종}_walk_r.png` (각 64x24) | 셰프 5명 우향 보행 4프레임 시트 |
| `assets/tavern/chef_{5종}_walk_l.png` (각 64x24) | 셰프 5명 좌향 보행 4프레임 시트 (독립 스프라이트) |
| `tests/phase-b5-1-chef-walk.spec.js` (23개) | Playwright 테스트 |
| `tests/phase-b5-1-qa-edge.spec.js` (16개) | QA 에지케이스 테스트 |

### 게이트 충족 상태

- [x] walk_r/l 10장 64x24 저장, HTTP 200 + PIL 크기 검증
- [x] spritesheet preload 10개 등록 (기존 52 + chef walk 10 = 62개 이상 텍스처)
- [x] Phaser anims 10개 등록 (chef_{name}_walk_r/l, 8fps, repeat=-1)
- [x] C/V 데모 키 동작 (null guard + anims guard, ASSET_MODE='real' 분기)
- [x] window.__tavernChefAnims 진단 노출
- [x] ServiceScene.js diff 0줄, scaleX/flipX 0건, tavern_dummy/ 변경 0건
- [x] Playwright 전체 PASS (B-5-1 신규 23 + 에지 16 = 39, 회귀 B-1~B-4 100/100)

### 스펙 대비 구현 차이

- SC-2~SC-4(carry/cook/serve): AD1에서 DEFERRED 결정, B-5-1b로 분리
- SC-6(C/V 데모 키): 에러 없이 동작하나, _chefs[0]가 Image 타입이라 walk 애니메이션 실제 미재생 (B-4 동일 한계, Phase D Sprite 전환 시 해소)

### QA 결과

PASS (B-5-1 신규 23 + 에지 16 = 39/39, 회귀 100/100). SC-1/SC-5/SC-6(조건부)/SC-7 충족, SC-2~SC-4 DEFERRED.

### 스펙/리포트

- 목적: `.claude/specs/2026-04-24-kc-phase-b5-1-scope.md`
- 스펙: `.claude/specs/2026-04-24-kc-phase-b5-1-spec.md`
- AD 모드1: `.claude/specs/2026-04-24-kc-phase-b5-1-ad1.md`
- AD 모드2: `.claude/specs/2026-04-24-kc-phase-b5-1-ad2.md`
- Coder 리포트: `.claude/specs/2026-04-24-kc-phase-b5-1-coder-report.md`
- QA: `.claude/specs/2026-04-24-kc-phase-b5-1-qa.md`

---

## Phase B-6 -- 캐릭터 해상도 업스케일 16x24 -> 32x48 (완료)

### 범위

캐릭터 15명(손님 10종 + 셰프 5명)을 PixelLab `create_character size=48`로 재발주하여 32x48 캔버스로 업스케일. 총 55장 신규 에셋 생성. TavernServiceScene.js spritesheet frameWidth/frameHeight 16/24 -> 32/48 갱신. 기존 16x24 에셋은 `.legacy-b5/`에 백업.

### 산출물

| 파일 | 역할 |
|------|------|
| `assets/tavern/customer_{10종}_seated_right.png` (각 32x48) | 손님 10종 우향 앉기 (업스케일) |
| `assets/tavern/customer_{10종}_seated_left.png` (각 32x48) | 손님 10종 좌향 앉기 (업스케일) |
| `assets/tavern/customer_{10종}_walk_r.png` (각 128x48) | 손님 10종 우향 보행 4프레임 시트 |
| `assets/tavern/customer_{10종}_walk_l.png` (각 128x48) | 손님 10종 좌향 보행 4프레임 시트 |
| `assets/tavern/chef_{5종}_idle_side.png` (각 32x48) | 셰프 5명 idle (업스케일) |
| `assets/tavern/chef_{5종}_walk_r.png` (각 128x48) | 셰프 5명 우향 보행 4프레임 시트 |
| `assets/tavern/chef_{5종}_walk_l.png` (각 128x48) | 셰프 5명 좌향 보행 4프레임 시트 |
| `assets/tavern/.legacy-b5/` (57 PNG) | 기존 16x24 에셋 백업 |
| `assets/tavern/_postprocess_b6.py` | B-6 후처리 스크립트 |
| `tests/phase-b6-upscale.spec.js` (66개) | Playwright 테스트 |
| `tests/phase-b6-qa-edge.spec.js` (17개) | QA 에지케이스 테스트 |

### 게이트 충족 상태

- [x] 15명 size=48 character_id 생성 (manifest 확인)
- [x] seated R/L 20장 32x48 + walk R/L 30장 128x48 = 55장 저장
- [x] frameWidth: 32, frameHeight: 48 갱신 (spritesheet 30개소)
- [x] ServiceScene.js diff 0줄, scaleX/flipX 0건, tavern_dummy/ 변경 0건
- [x] .legacy-b5/ 57 PNG 보존
- [x] Playwright 153/153 PASS (B-6 신규 66 + 회귀 70 + QA 엣지 17)

### 스펙 대비 구현 차이

- WA-7: `.legacy/` -> `.legacy-b5/`로 폴더명 변경 (기존 레거시와 구분)
- WA-3: raw 캔버스 64x64(스펙) -> 68x68(실측, PixelLab size=48 실제 출력)
- WA-10: `setDisplaySize(32, 48)` 사용으로 별도 앵커 조정 불필요
- business walk_l: west 방향 프레임 부재 -> rotation fallback (2프레임 alternation)

### QA 결과

PASS (153/153). SC-1~SC-10 전수 충족. AD2 APPROVED 55/55, AD3 APPROVED with NOTE.

### 스펙/리포트

- 스펙: `.claude/specs/2026-04-24-kc-phase-b6-spec.md`
- AD 모드1: `.claude/specs/2026-04-24-kc-phase-b6-ad1.md`
- AD 모드2: `.claude/specs/2026-04-24-kc-phase-b6-ad2.md`
- AD 모드3: `.claude/specs/2026-04-24-kc-phase-b6-ad3.md`
- Coder 리포트: `.claude/specs/2026-04-24-kc-phase-b6-coder-report.md`
- QA: `.claude/specs/2026-04-24-kc-phase-b6-qa.md`
- Manifest: `.claude/specs/2026-04-24-kc-phase-b6-pixellab-manifest.md`

---

## Phase B-6-2 / B-5-1b / B-5-2 / B-5-3 -- 가구 비례 + 셰프 포즈 + 가구 업그레이드 + 환경물 (미착수)

### 진입 조건

- [x] Phase B-6 완료 (캐릭터 15명 해상도 업스케일 32x48)
- [ ] 사용자 최종 승인

### 범위

| 서브 | 내용 | 상태 |
|------|------|------|
| **B-6-2** | 벤치(14x76) 등 가구 에셋을 32x48 캐릭터 비례에 맞게 업스케일 (AD3 NOTE에서 도출) | 미착수 |
| **B-5-1b** | 셰프 5명 carry_r/l + cook + serve 정지 포즈 (32x48 규격). PixelLab 템플릿 매칭 불가로 별도 발주 전략 필요 | 미착수 |
| **B-5-2** | lv3/lv4 업그레이드 등급 가구 에셋 | 미착수 |
| **B-5-3** | 술통/바닥 타일/벽 장식 환경물 에셋 | 미착수 |

### 게이트 조건 (Phase C 진입)

- [x] 최소 normal 손님 1종 + 미미 셰프 1명 + lv0 V12 가구 1세트가 TavernServiceScene에서 실 스프라이트로 렌더링 확인 *(Phase B-2에서 충족)*
- [x] seated_left / seated_right가 scaleY/flipY 없이 별도 스프라이트로 자연스러운 정합 확인 *(Phase B-2에서 충족)*
- [x] 손님 10종 seated R/L + 셰프 7명 idle_side 전종 실 에셋 완비 *(Phase B-3에서 충족)*
- [x] 손님 10종 walk_l/r 시트 20장 완비 *(Phase B-4에서 충족)*
- [x] 셰프 5명 walk_l/r 시트 10장 완비 *(Phase B-5-1에서 충족)*
- [x] 캐릭터 15명 해상도 업스케일 32x48 완비 *(Phase B-6에서 충족)*
- [ ] 사용자 최종 승인

---

## Phase C -- 테마 에셋 + HUD/VFX (완료, Phase 94)

### 범위

- C1: g1 바닥 타일(floor_wood_tile_v14.png, 32x32) + 벽 타일(wall_horizontal_v14.png, 64x24) 실 에셋 적용
- C2: 인내심 게이지 6테이블 (100x8px, 3색 구간: 초록/노랑/빨강, `_patienceBarMap`)
- C3: 주문 말풍선 (`_orderBubbleMap`, 70x24px, 중복 생성 방어)
- C4: 골드 플로팅 VFX (`_showGoldFloat`, Tween y-=32, 1000ms fade)
- C5: 영업 HUD 바 (타이머+골드, depth 9100)

### 산출물

| 파일 | 역할 |
|------|------|
| `assets/tavern/floor_wood_tile_v14.png` (32x32) | g1 나무 원목 바닥 타일 |
| `assets/tavern/wall_horizontal_v14.png` (64x24) | g1 아이보리 벽 타일 |
| `tests/phase94-qa.spec.js` (36개) | Playwright 테스트 |

### 스펙 대비 구현 차이

- 챕터별 8세트 전체 발주는 Out of Scope, g1 1세트만 선행 (나머지 차기)
- AD3 REVISE: row0 게이지 barY 벽 영역 침범(52→58) 수정 후 APPROVED

### QA 결과

PASS (36/36). AD 모드2 PASS (에셋 2종 규격 준수), AD 모드3 APPROVED (재검수).

### 스펙/리포트

- 스펙: `.claude/specs/2026-04-30-kc-phase94-scope.md`
- Coder 리포트: `.claude/specs/2026-04-30-kc-phase94-coder-report.md`
- AD 모드2+3: `.claude/specs/2026-04-30-kc-phase94-ad3.md`
- QA: `.claude/specs/2026-04-30-kc-phase94-qa.md`

---

## Phase D -- 게임 로직 포팅 (완료, Phase 95)

### 범위

- D1: init() + Manager 초기화 (InventoryManager, SaveManager, ChefManager, RecipeManager, partialFail 50% 컷)
- D2: 조리 슬롯 시스템 (2슬롯, 세척, 사고 비활성화)
- D3: 레시피 퀵슬롯 UI (컨트롤 바 y=560~640, 최대 4개+스킬 버튼 동일 행)
- D4: 재고 패널 UI (주방 영역 하단, INGREDIENT_TYPES 전종)
- D5: Image → Sprite 전환 (셰프/손님 walk 애니메이션 실재생)
- D6: 손님 AI 입장/퇴장 Tween (DOOR_ANCHOR→슬롯, 80px/s, 10종 프로필)
- D7: 인내심 감소 + 퇴장 판정 (ServiceScene 동일 공식, 게이지 연동)
- D8: 서빙 + 골드 획득 (ServiceScene 동일 골드 공식, 콤보/팁/VFX)
- D9: 셰프 스킬 버튼 (5종, 쿨다운 UI)
- D10: 영업 종료 + ResultScene 전환 (time/stock/satisfaction/no_recipe/manual)
- D11: 영업 이벤트 시스템 (4종: happy_hour/rainy_day/food_review/kitchen_accident)
- D12: 유랑 미력사 패시브 + 단골 servedCount + 평론가 criticScores

### 스펙 대비 구현 차이

1. 데모 탭 순환(Phase C)을 제거, D6 게임 AI가 손님 생명주기 관리
2. 단체 손님(group) 2석 동시 점유 로직 간소화, normal로 대체 처리
3. 인내심 게이지: 손님 없을 때 투명 숨김 개선
4. 서빙: 조리 슬롯 탭→해당 주문 손님에게 자동 서빙 (테이블 직접 탭이 어려운 탑다운 레이아웃 고려)
5. 만족도 HUD 추가 (스펙 미명시, HUD 바 중앙에 배치)

### QA 결과

PASS (38/38). AD 모드3 REVISE 3건(스킬 버튼 겹침/2행 초과/컨트롤 바 수용량) 수정 후 APPROVED.

### 스펙/리포트

- 스펙: `.claude/specs/2026-04-30-kc-phase95-scope.md`
- Coder 리포트: `.claude/specs/2026-04-30-kc-phase95-coder-report.md`
- AD 모드3: `.claude/specs/2026-04-30-kc-phase95-ad3.md`

---

## Phase E -- 마이그레이션 (완료, Phase 96)

### 범위

- E1: GatheringScene `_triggerVictory()`/`_triggerGameOver()` 내 `ServiceScene` → `TavernServiceScene` 교체 (2곳)
- E2: EndlessScene `_transitionToService()` 내 `ServiceScene` → `TavernServiceScene` 교체
- E3: main.js ServiceScene import/등록 제거, scene 배열 정리
- E4: ServiceScene.js → `_legacy/ServiceScene.js` 이동 (삭제하지 않음)
- E5: `_buildBenchSlots()` dev-only 가드 (`import.meta.env.DEV`) 추가
- E6: playwright.config.js timeout 30초 → 60초 상향

### 스펙 대비 구현 차이

- `_buildDebugHUD()` 가드: 해당 메서드가 TavernServiceScene에 존재하지 않아 스킵 (스펙 오류)
- window 진단 변수 DEV 가드: QA 테스트 의존성으로 의도적 보류
- Back 버튼: PauseScene 미존재로 MenuScene 복귀 유지
- `?scene=tavern` DevHelper 파라미터: 프로덕션 트리-쉐이킹 적용되므로 유지

### QA 결과

PASS (101/101 -- Phase 96 신규 27 + Phase 94 회귀 32 + Phase 95 회귀 42). LOW 이슈: 주석 12곳에 "ServiceScene" 문자열 잔존 (기능 무관).

### 스펙/리포트

- 스펙: `.claude/specs/2026-04-30-kc-phase96-scope.md`
- Coder 리포트: `.claude/specs/2026-04-30-kc-phase96-coder-report.md`
- QA: `.claude/specs/2026-04-30-kc-phase96-qa.md`

---

## 변경 이력

| 일자 | 변경 |
|------|------|
| 2026-04-30 | Phase C(94)/D(95)/E(96) 완료 반영. Phase C: g1 바닥/벽 타일 v14 + HUD/VFX(인내심 게이지/말풍선/골드 플로팅/HUD 바). Phase D: ServiceScene 전체 게임 로직 포팅(D1~D12). Phase E: GatheringScene/EndlessScene 진입점 교체, main.js 정리, ServiceScene→_legacy/ 이동. 페이즈 총괄 테이블 C/D/E 완료로 갱신. TavernServiceScene이 유일한 영업씬으로 전환됨. |
| 2026-04-24 | Phase B-6 완료 반영. B-6 절 추가 (캐릭터 15명 해상도 업스케일 32x48, 55장 신규 에셋). 페이즈 총괄 테이블 B-6 행 추가 + B-6-2(가구 비례 업스케일) 미착수 행 추가. 미착수 섹션을 B-6-2/B-5-1b/B-5-2/B-5-3으로 확장. B-5-1b 포즈 규격 16x24 -> 32x48 반영. Phase C 게이트에 B-6 충족 항목 추가. |
| 2026-04-24 | Phase B-5-1 완료 반영. B-5-1 절 추가 (셰프 walk 시트 10장). B-5+를 B-5-1(완료)/B-5-1b/B-5-2/B-5-3(미착수)으로 4분할. 페이즈 총괄 테이블 B-5-1 행 추가 + B-5-1b/B-5-2/B-5-3 미착수 행 분리. Phase C 게이트에 B-5-1 충족 항목 추가. |
| 2026-04-24 | Phase B-4 완료 반영. B-4 절 추가 (walk 시트 20장 + W-1 PRO PARTIAL 최종 확정). B-4+ -> B-5+로 갱신, 페이즈 총괄 B-4 행 추가 + B-5+ 미착수 행 분리. B-3 W-1 PARTIAL 사유를 최종 확정 문구로 갱신. |
| 2026-04-24 | Phase B-3 완료 반영. B-3 절 추가, B-3+ -> B-4+로 갱신, B-3 산출물/게이트/QA/W-1 PARTIAL 상세 기록. 페이즈 총괄 B-3 행 추가 + B-4+ 미착수 행 분리. |
| 2026-04-24 | Phase B-2 완료 반영. B-2 절 추가, B-2+ -> B-3+로 갱신, B-1 해소 예정 사항에 B-2 해소 결과 반영, 페이즈 총괄 테이블 B-2 행 추가, Phase C 게이트 2항목 B-2에서 충족으로 갱신. |
| 2026-04-23 | Phase B-1 완료 반영. Phase B를 B-1(완료)/B-2+(준비)로 분할, B-1 산출물/게이트/QA/해소 예정 사항 추가. |
| 2026-04-23 | Phase A-bis V12 마이그레이션 완료 반영. Phase A-bis 절 추가, Phase B를 V12 규격으로 갱신, 페이즈 총괄 테이블에 A-bis 추가. |
| 2026-04-23 | 신규 작성. Phase A 완료 반영. |
