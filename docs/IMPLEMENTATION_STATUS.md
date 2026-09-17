# 구현 및 검증 상태

마지막 갱신일: 2026-09-16

## 마일스톤

| 단계 | 상태 | 근거 |
|---|---|---|
| M0 문제·데이터·모델 계약 | 로컬 CPU 통과·사용자 GPU 확인 대기 | 사양·데이터·모델 문서, 5 tests, smoke 통과 |
| M0.5 외부 원본 수집 | 완료 | 원본 다운로드·checksum·schema 감사·PNG 완료 |
| M1 축대칭 전기 solver | 완료 | 해석해·전류보존·Joule scaling 테스트와 평가 그림 |
| M2 과도 열 solver·연성 | 완료 | backward-Euler 열전달·에너지보존·수렴 테스트와 평가 그림 |
| M2.1 enthalpy 상변화 solver | 수치 구현 완료·재료 보정 대기 | 11 gates PASS, 해석해·M2 회귀·에너지·PNG 통과 |
| M2.2 온도의존 전기–열 약연성 | 수치 구현 완료·고온 물성 BLOCK | 15 gates PASS, frozen 회귀·전류·교차에너지·PNG 통과 |
| M2.3 고온 벌크저항 민감도 | 완료·접촉/강종 BLOCK | 9 gates PASS, NBS 직접점 보존·3 bridge·상변화·PNG 통과 |
| M2.4 명시적 전기 접촉저항 | 수치 구현 완료·보정 BLOCK | 12 gates PASS, 해석해·0-contact 회귀·보존형 접촉열·PNG 통과 |
| M2.5 상태의존 접촉법칙·식별성 | 수치 계약 완료·stack 보정 BLOCK | 15 gates PASS, T/F law·rank 1 식별성·외부 관측성·PNG 통과 |
| M2.6 목표 stack·bounded preflight | 완료·V3 release BLOCK | 11 gates PASS, 359 stack-compatible, force overlap 0, 19-case OAT·PNG 통과 |
| M2.7 단위·calibration feasibility | 완료·physical twin BLOCK | 16 gates PASS, N/PSI 충돌·sensor chain·후보 5종·PNG 6종 감사 |
| M3 simulation Dataset | 완료 | 결정론적 64/16/16/16/8 split, manifest·누수·checksum 검증 |
| M3.1 space-filling Dataset | 전체 생성·무결성 검증 완료 | RSW-SIM-V2 1,920 sample, checksum·누수·보존 PASS |
| M3.2 물리 타당성 gate | 완료·전체범위 학습 BLOCK | 분포 PASS, 1700 K 초과 train 7개, 상변화 모델 필요 |
| M4 기준 모델 비교 | mean·3D CNN 완료 | 64-sample overfit gate, validation 선택, JSON·CSV·PNG 구현 |
| M5 PhysicsNeMo operator 비교 | 완료 | FNO gate PASS, Transolver gate FAIL |
| M5.1 FNO physics constraint | 완료 | reference gate PASS, 4-way 100-epoch ablation 완료 |
| M5.2 frozen test·OOD 평가 | 완료 | SHA-256 동결, IID·process·corner 보고용 평가와 PNG 완료 |
| M6 외부 데이터 adapter | 완료 | 두 독립 manifest·checksum·group split·conflict flag·PNG 완료 |
| M6.1 외부 CTQ 기준선 | 완료 | 20 gates PASS, 315/78 split, 3 model hash 동결·PNG 7종 |
| M6.2 frozen external test | 완료·test consumed | 27 gates PASS, 98 eligible, bootstrap 5,000회·PNG 7종 |
| M7.1 개발 품질 gate·OOD | 완료·production BLOCK | nested 5×4 CV, 14 gates PASS·PNG 7종 |
| M7.2 Polito Fault 보조 track | 완료·성능 release FAIL | test PR-AUC 0.4157, Fault recall 0.3333 |
| M8 mesh·최종 보고서 | 대기 | 비정형 mesh가 있을 때만 MGN 활성화 |

## M0 조사 결과

- Search 스킬로 검색 결과 65건을 검토했다.
- 공식 문서상 FNO, Transolver, MeshGraphNet, PINO/PINN, DoMINO를 비교했다.
- 현재 Conda 환경은 `nvidia-physicsnemo==2.1.1`이다.
- FNO, Transolver, MeshGraphNet, DoMINO, `physicsnemo.sym` import를 확인했다.
- GeoTransolver는 현재 버전에 없어 후보에서 보류했다.
- 작은 3D FNO와 point-token Transolver의 forward/backward를 직접 확인했다.
- Transolver의 3D `structured_shape` 경로는 2.1.1에서 실패해 명시 좌표 path로 고정했다.
- CPU 검증에서 `5 passed`, FNO/Transolver의 출력 shape과 모든 parameter gradient가
  정상임을 확인했다.

## 사용자 확인 명령

```bash
conda activate physicsnemo-study
python scripts/smoke_rsw_model_candidates.py
python -m pytest tests/test_rsw_model_candidates.py -q
ruff check src/physicsnemo_study/models/rsw_candidates.py \
  scripts/smoke_rsw_model_candidates.py tests/test_rsw_model_candidates.py
```

예상 결과는 FNO와 Transolver 모두 channels-first 기준 동일한 출력 shape을 내고, 모든
학습 parameter의 gradient가 finite하다는 JSON이다. 이 결과는 정확도 비교가 아니다.

## M5.1 현재 결과

- M3 train·validation 정답장에 M2와 동일한 differentiable residual을 적용했다.
- 최악의 one-step equivalent temperature residual은 `3.510684e-4 K`로 사전 기준
  `1e-3 K`를 통과했다.
- 초기조건 최대 오차는 `0 K`다.
- `data_only → +initial → +nonnegative → +thermal_residual`의 누적 ablation runner를
  구현했다.
- 실제 PhysicsNeMo FNO3D CUDA 1-epoch path check에서 checkpoint, 평가 JSON·CSV와 그림 네
  장 생성을 확인했다. 1 epoch 결과는 성능 판정에 사용하지 않는다.

## M5.2 결론

- 사전 규칙에 따라 `data_only` FNO epoch 20을 SHA-256으로 동결했다.
- test IID Tmax MAE는 `48.112 K`다.
- process OOD와 corner OOD Tmax MAE는 각각 `319.588 K`, `426.726 K`다.
- 고온 OOD 과소예측이 확인되어 현재 smoke surrogate를 품질 판정에 사용하지 않는다.
- test 결과를 사용한 E05 재선택·재튜닝은 금지한다.

## M6 결론

- Mendeley 4,186개 반복 record를 495개 독립 `sample_id` weld로 집계했다.
- development/test는 396/99 weld이며 Good/Bad/Explode를 계층화했다.
- 서로 다른 정적 값이 보고된 5개 weld의 scalar를 비워 두고 min/max/충돌 flag를 보존했다.
- Polito 1,976행은 불변 `sample_row_id`로 row-order 결합했다.
- `Car Body` group split은 1,380/300/296행이고 fault 수는 55/12/12다.
- 두 manifest 모두 checksum과 ID 재검증을 통과했고 group leakage는 0이다.
- 두 외부 데이터가 내부 온도장 label이 아니라는 금지 계약을 manifest에 기록했다.

## M3.1/M3.2 결론

- RSW-SIM-V2 manifest ID는
  `2e43da1c058aef17a5500145fc92ea241b038457973adbcddb7162c1b39d2b98`다.
- 모든 checksum, group leakage, tensor schema와 에너지 보존 검사는 통과했다.
- train-validation Tmax/energy KS는 `0.0771`/`0.0674`로 sampling gate를 통과했다.
- 1700 K 초과는 train 7개, process OOD 12개, corner OOD 21개다.
- V2 생성기는 상변화·온도의존 물성이 없으므로 전체범위 FNO 재학습은 차단했다.
- test 열람은 사전 dataset 물리범위 감사로 한정하며 모델 선택에는 사용하지 않는다.


## M2.2 결론

- interval-lagged 방식으로 각 열 시간구간 시작 온도에서 전기전도도와 전기장을 갱신한다.
- 상수 전기물성 단방향 회귀의 최대 온도 차이는 `1.705e-13 K`다.
- 온도의존/frozen 최고온도는 `634.869/575.021 K`다.
- 온도의존/frozen terminal energy는 `77.507/51.468 J`다.
- 전류 보존, 목표 전류, terminal–Joule energy와 enthalpy 수지는 모두 gate를 통과했다.
- M2.3에서 bulk curve를 액상선까지 연결했으며 강종·접촉저항 미보정으로 V3 차단을 유지한다.

## M2.3 결론

- NBS/NIST의 300–1000 K 및 1500–1800 K 철 전기저항 자료를 provenance와 함께 고정했다.
- 직접 점이 없는 1000–1500 K는 early/linear/late 세 endpoint-preserving bridge로 분리했다.
- 세 scenario의 Tmax/energy/액상분율 범위는 `0.742 K`/`9.191 J`/`0.025583`이다.
- 액상분율은 민감했지만 10-cell 반경격자의 mushy 직경은 모두 `3.0 mm`로 양자화됐다.
- 80→160 time-step Tmax 차이는 `3.789 K`로 사전 기준 `5 K`를 통과했다.
- bulk 온도범위는 액상선까지 연결됐지만 강종·접촉저항 보정이 없어 V3는 차단한다.

## M2.4 결론

- axial 내부 face와 두 전극 경계에 정적 면적비 접촉저항 `Rpp [Ω·m²]` API를 추가했다.
- `Rpp=0` 전위/전류 회귀 및 `R=L/(σA)+Rpp/A` 해석해를 통과했다.
- 접촉 `I²R`을 인접 열 cell에 보존적으로 배분하고 전극 흡수 몫을 별도 기록했다.
- 12/12 gate, 80→160 time-step 최고온도 차이 `0.474 K`, PNG 5종을 통과했다.
- nominal에서 terminal energy의 84.20%가 합성 contact resistance에서 발생했다.
- 이 큰 비율은 보정 결과가 아니라 기존 유효저항을 분해한 초기값의 민감도다.
- low/nominal/high Tmax는 `807.13/1288.80/1768.18 K`로 예상 순서를 보였다.

## M2.5 결론

- `Rpp(T,F,stack,surface)` power-law protocol, provenance와 fail-closed 유효범위를 구현했다.
- 온도·force exponent가 0이면 M2.4와 온도·전압·에너지가 정확히 일치한다.
- 2.0/3.5/5.0 kN 합성 force 조건의 Tmax는 `1282.27/1034.11/911.47 K`다.
- 세 자유 접촉항의 관측행렬은 rank 1, nullity 2이므로 단자 V/I만으로 분리할 수 없다.
- 같은 총 접촉저항에서도 sheet source power가 46.2% 달라질 수 있음을 보였다.
- Mendeley는 전압이 없고 Polito는 SI scaling이 없어 절대·계면별 Rpp 보정을 차단했다.
- 15/15 gate, 60→120 time-step Tmax 차이 `2.050 K`, PNG 6종을 통과했다.

## M2.6 결론

- target은 `AISI1010_0p63x2_surfaceUnknown_CuAlloyTip3p175`로 고정했다.
- Mendeley development 396개 중 material match 396개, thickness-compatible 361개, conflict-free stack-compatible 359개다.
- 기존 1.5–6.0 kN contact-law 유효범위와 force가 겹치는 development weld는 0개다.
- 공개 자료가 말하지 않는 coating/surface와 effective electrical contact radius는 unknown으로 유지했다.
- 전기·열·상변화·접촉 입력 12개를 fixed/bounded/nuisance와 source status로 분류했다.
- 19-case OAT baseline Tmax/energy는 `1327.954 K`/`110.919 J`이고, 전체 최고온도는 `1774.719 K`다.
- 가장 큰 Tmax span은 faying resistance `899.705 K`, current `466.500 K`, contact radius `442.462 K`다.
- 최대 수치보존 상대오차는 `8.283e-13`, 11/11 verification gate는 PASS다.
- 이는 차단을 포함한 preflight 성공이며 `v3_generation_ready=false`다.

## M2.7 결론

- 같은 force signal이 CSV/Data in Brief에서는 N, Sensors Table 5에서는 PSI로 표기된다.
- N을 가정한 기존 overlap 0은 조건부 수치이며 물리 overlap 상태는 `indeterminate`다.
- 0–2128 g 질량 9점 검사는 RMSE `2.828 g`이지만 electrode transfer calibration은 아니다.
- development 396 weld/3,378 records를 test outcome 없이 감사했다.
- 비양수·결측 current/force 1/14 records를 대체하지 않고 flag로 보존했다.
- 134 weld는 nominal 10 Hz 양끝 포함 표본수 가정과 맞지 않아 timestamp를 생성하지 않았다.
- current Jensen ratio 중앙값은 `1.029602`; 평균값이 변동을 잃지만 energy로 해석하지 않는다.
- 공개 후보 5종 중 SI terminal-energy 보정 및 내부 온도장 검증 가능 후보는 각각 0개다.
- lab CTQ 기준선은 가능하되 force를 기본 feature에서 제외한다.
- physical twin과 RSW-SIM-V3는 계속 차단하고 27 calibration + 15 qualification coupon의
  학습용 최소 protocol을 문서화했다.
- audit manifest ID는
  `8394d8ce236de1b3a7c75fd346806f731627ef6e2eab2c2b5d4c7d859df9774d`다.

## M6.1 결론

- external test outcome 없이 development 396 weld만 읽고 feature-eligible 393개를 만들었다.
- sample ID 기준 internal train/validation은 315/78이며 overlap은 0이다.
- N/PSI 충돌 force와 세 CTQ target은 feature에서 차단했다.
- nugget random forest는 MAE `0.2324 mm`, R² `0.3752`로 mean MAE보다 21.22% 낮다.
- pull-test random forest는 MAE `174.15 N`, R² `0.6829`로 mean MAE보다 42.30% 낮다.
- category random forest는 macro-F1 `0.6813`, balanced accuracy `0.6175`다.
- category recall은 Bad/Explode/Good `0.667/0.200/0.986`으로 Explode 검출이 부족하다.
- 높은 accuracy `0.9231`만으로 품질 모델을 승인하지 않으며 자동 합격에는 사용하지 않는다.
- split과 model 3개의 SHA-256을 selection manifest
  `174006ad63c751f58f40a38b8a4c27dbf7f110727f4f0c854b0f29801be6fd01`로 동결했다.
- development-only fail-closed 접근과 저장 모델 재로딩 예측 검증을 포함한 20/20 gate 및
  PNG 7종을 통과했고 physical twin 및 V3 차단은 유지한다.

## M6.2 결론

- 사전 protocol과 selection/model/data hash를 확인한 뒤 external test 99 weld를 평가했다.
- current communication error 1 weld를 제외해 nugget/category 98개, pull-test 97개다.
- nugget MAE/R²는 `0.2368 mm / 0.2055`, pull-test는 `147.62 N / 0.5830`이다.
- category accuracy/macro-F1/balanced accuracy는 `0.8878/0.6003/0.5682`다.
- Bad/Explode/Good recall은 `0.750/0.000/0.955`; Explode 6개를 모두 Good으로 놓쳤다.
- weld bootstrap 5,000회, subgroup 결과와 PNG 7종을 생성했고 27/27 gate를 통과했다.
- 모델 재학습·재선택·threshold 조정은 하지 않았고 자동 합격은 계속 차단한다.
- evaluation receipt ID는
  `4dbee4092cde82bc3735a0f055fde8ff7a041dd15cf7efd237de27cd29d5362b`다.

## M7.1 결론

- Mendeley development feature-eligible 393 weld만 사용해 5 outer × 4 inner nested
  group cross-fitting을 수행했다.
- M6.2 external test outcome은 모델·보정·임계값·OOD 규칙 선택에 재사용하지 않았다.
- raw→보정 failure Brier는 `0.07336→0.06039`, ECE는 `0.07676→0.02700`으로 개선됐다.
- 반면 3-class balanced accuracy는 `0.66406→0.60594`, Explode recall은 `0.20→0`으로
  감소해 확률 calibration과 class discrimination의 차이를 확인했다.
- 결합 진단 정책은 false escape `1/42=2.38%`, false reject `0/351=0%`지만 review
  `359/393=91.35%`, auto coverage `8.65%`로 지나치게 보수적이다.
- 결합 OOD 후보는 18개(4.58%)를 flag했지만 알려진 OOD label이 없어 recall·detector
  우열을 주장하지 않는다.
- 2,000회 category-stratified bootstrap, weld별 CSV, JSON, 해석 문서, PNG 7종과 구현·산출물
  hash manifest를 생성했다.
- artifact manifest ID는
  `2b9f0110d10d962aab74377bd04915236439ea24a48f2c326a057622ef3330ca`다.

## M7.2 결론

- Polito train/validation 1,380/300행과 Car Body 20/4개로 test-sealed 개발을 수행했다.
- prior, balanced logistic, balanced Random Forest, 1D CNN을 비교해 validation PR-AUC
  `0.6104`의 Random Forest를 선택했다.
- upstream normalized 신호에서 padding-aware 통계 feature 69개를 만들고 Car Body·날짜·
  Welding Spot·row ID를 model feature에서 제외했다.
- Force 유효 길이 0은 train 166개, validation 63개, test 55개이며 보간하지 않고 mask와
  길이 0으로 유지했다.
- M7.2a validation triage는 fault escape 0, review 56.33%, auto coverage 43.67%였다.
- 모델·feature·threshold·evaluator·runner를 hash로 고정한 뒤 test 296행을 한 번 평가했다.
- Frozen test PR-AUC/ROC-AUC는 `0.4157/0.7994`, Fault recall/precision은
  `0.3333/0.4000`이다.
- 동결 triage는 Fault 12개 중 NORMAL 2, REVIEW 8, FAULT 2로 false escape `16.67%`,
  review `54.39%`, auto coverage `45.61%`다.
- 무결성 gate는 모두 통과했지만 false escape 때문에 performance/production release는 실패다.
- test 기반 재학습·재선택·threshold 조정은 수행하지 않았고 새 holdout 없이는 개선 모델을
  승인하지 않는다.
- M7.2a selection manifest ID는
  `2ca70b27dc8420683d9914cf2882199c9b9cdff6b6fb8e79bf4a79545612b653`, M7.2b evaluation
  receipt ID는 `0c115d373eb8e76a79d59817f548ac046ee03a3719ae1906cf3daf91eda382c5`다.

## 다음 gate

M8에서는 2026-09-18 포트폴리오 마감을 위해 문제–물리 solver–PhysicsNeMo operator–외부 CTQ–
공장 Fault–fail-safe gate를 하나의 최종 보고서로 연결한다. 새 모델 실험은 중단하고 실행 명령,
핵심 그림, 성공/실패 결과와 지원 직무 연결을 정리한다.

자동차 physical-twin track은 traceable SI V/I/F와 stack/contact metadata가 생기기 전까지
차단한다. RSW-SIM-V3도 만들지 않으며 M2.1–M2.7 proxy와 동결 V2 test 결과를 변경하지 않는다.
