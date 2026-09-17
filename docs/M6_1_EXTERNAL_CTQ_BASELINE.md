# M6.1 development-only 외부 CTQ 기준선

## 목적

M6.1은 Mendeley의 실험실 점용접 데이터에서 공정 입력만으로 다음 CTQ를 어느 정도
예측할 수 있는지 확인하는 통계 기준선이다.

- nugget diameter `[mm]`
- pull-test force `[N]`
- category `Bad / Explode / Good`

이 모델은 PhysicsNeMo의 내부 온도장 surrogate가 아니다. 간단한 모델을 먼저 평가하는
이유는 이후 시뮬레이션 CTQ나 PhysicsNeMo feature를 추가했을 때 단순 공정변수보다 실제로
개선됐는지 비교하기 위해서다.

## 데이터 접근 계약

M6에서 미리 만든 external split 중 `development` 396 weld만 읽었다. 외부 test 99 weld의
입력·정답은 모델 학습, 선택, threshold 설정과 결과 그림에 사용하지 않았다.

development 안에서 feature-eligible weld를 category별 seed 42로 다시 나눴다.

| internal split | Bad | Explode | Good | 합계 |
|---|---:|---:|---:|---:|
| train | 14 | 20 | 281 | 315 |
| validation | 3 | 5 | 70 | 78 |

각 행은 하나의 독립 `sample_id` weld이고 train-validation overlap은 0이다. 이 분할은 random
interpolation 평가다. 새로운 판재 lot, 설비 또는 생산공장 OOD 성능을 뜻하지 않는다.

## feature 계약

사용한 12개 feature는 다음과 같다.

| 종류 | feature | 의미 |
|---|---|---|
| 설정 | `pressure_psi` | 공압 설정값; electrode tip force가 아님 |
| 설정 | `welding_time_ms`, `angle_deg` | 공개 static process setting |
| stack | `thickness_a_mm`, `thickness_b_mm` | 두 판재 두께 |
| acquisition | `record_count` | weld 안에 공개된 record 수; 실제 시간축이 아님 |
| current | positive mean, RMS, min, max | 비양수 sentinel을 제외한 current 요약 |
| current | `current_jensen_ratio` | `mean(I²)/mean(I)²`; 에너지가 아님 |
| quality flag | `invalid_current_count` | 제외한 비양수 current 수 |

`force_n`, force min/mean/max는 M2.7의 N/PSI 충돌 때문에 금지했다. 세 target도 feature에
들어가지 않는다. timestamp를 만들거나 current를 Joule energy로 변환하지 않았다.

feature complete-case는 393 weld다. 다음 세 weld는 대체하거나 임의 보간하지 않았다.

- 117, 325: 판재 두께 충돌로 scalar가 비어 있음
- 252: 유효한 양의 current record가 없음

nugget/category는 393개, pull-test는 충돌값 2개를 추가 제외해 391개가 평가 대상이다.

## 고정 모델 후보

validation을 보고 hyperparameter 탐색을 반복하지 않도록 후보와 설정을 YAML에 미리
고정했다.

### 회귀

- `mean`: train target 평균을 항상 출력
- `ridge`: 표준화 + L2 선형회귀
- `random_forest`: 깊이와 leaf 크기를 제한한 비선형 ensemble

### 분류

- `prior`: 가장 흔한 train class를 출력
- `logistic_balanced`: class-weighted 다중 logistic regression
- `random_forest_balanced`: class-weighted random forest

회귀는 validation MAE가 가장 작은 후보, 분류는 macro-F1이 가장 큰 후보를 선택했다.

## metric의 의미

회귀 오차는 다음과 같다.

\[
\mathrm{MAE}=\frac{1}{N}\sum_i|\hat y_i-y_i|,
\qquad
\mathrm{RMSE}=\sqrt{\frac{1}{N}\sum_i(\hat y_i-y_i)^2}
\]

RMSE는 큰 오차에 더 민감하다. 결정계수는

\[
R^2=1-\frac{\sum_i(y_i-\hat y_i)^2}{\sum_i(y_i-\bar y)^2}
\]

이며 1에 가까울수록 좋고, 0은 대략 평균 예측 수준, 음수는 평균보다도 나쁜 결과다.

분류에서 accuracy는 Good 351개가 많은 현재 데이터에서 오해를 일으킬 수 있다. 각 class
recall의 평균인 balanced accuracy와 class별 F1의 평균인 macro-F1을 같이 사용한다.

\[
\mathrm{BalancedAcc}=\frac{1}{C}\sum_{c=1}^{C}\mathrm{Recall}_c
\]

## validation 결과

### 회귀

| target | model | MAE | RMSE | R² |
|---|---|---:|---:|---:|
| nugget `[mm]` | mean | 0.2950 | 0.4151 | -0.0001 |
|  | ridge | 0.2505 | 0.3559 | 0.2646 |
|  | **random forest** | **0.2324** | **0.3281** | **0.3752** |
| pull test `[N]` | mean | 301.84 | 553.83 | -0.0066 |
|  | ridge | 217.59 | 370.81 | 0.5488 |
|  | **random forest** | **174.15** | **310.84** | **0.6829** |

선택 모델은 mean 대비 MAE를 nugget에서 `21.22%`, pull test에서 `42.30%` 줄였다. nugget
산점도는 큰/작은 값을 평균 쪽으로 당기는 경향이 남아 있어 고정밀 nugget predictor라고
보기 어렵다.

### category

| model | accuracy | balanced accuracy | macro-F1 |
|---|---:|---:|---:|
| prior | 0.8974 | 0.3333 | 0.3153 |
| logistic balanced | 0.5769 | 0.6127 | 0.4052 |
| **random forest balanced** | **0.9231** | **0.6175** | **0.6813** |

선택 모델의 recall은 Bad `0.667`, Explode `0.200`, Good `0.986`이다. 높은 accuracy에도
Explode 5개 중 4개를 Good으로 놓쳤으므로 품질 gate나 자동 합격 모델로 사용할 수 없다.
logistic 모델은 Explode recall 0.60이지만 false positive가 많아 macro-F1이 낮다. 이
trade-off는 M7의 `PASS / REVIEW / FAIL` 비용 민감 정책에서 별도로 다룬다.

0.60–0.66 mm validation subgroup은 Bad 1, Explode 4, Good 65개뿐이다. subgroup metric은
표본이 너무 작아 방향 확인 이상으로 해석하지 않는다.

## permutation importance

선택 모델의 validation feature를 하나씩 섞어 score가 얼마나 나빠지는지 20회 반복했다.
상위 변수는 다음과 같다.

- nugget: `record_count`, `welding_time_ms`, `thickness_a_mm`
- pull test: `record_count`, `welding_time_ms`, `current_jensen_ratio`
- category: `welding_time_ms`, `current_jensen_ratio`

`record_count`와 `welding_time_ms`는 강하게 연결될 수 있다. permutation importance는 상관된
변수 사이에서 중요도를 나눠 가지며 인과관계나 물리법칙을 증명하지 않는다. 음수 importance는
그 변수가 유용하다는 근거가 아니다.

## 동결 결과와 다음 단계

선택 모델 세 개는 pickle로 저장하고 SHA-256을 selection manifest에 기록했다. selection
manifest ID는
`174006ad63c751f58f40a38b8a4c27dbf7f110727f4f0c854b0f29801be6fd01`다.

다음 M6.2는 이 manifest와 model hash가 정확히 일치할 때만 외부 test를 한 번 읽는 별도
runner다. test 결과로 모델이나 hyperparameter를 다시 고르면 안 된다. test가 나빠도 먼저
실패 분석을 기록하고 새로운 실험 버전을 만들어야 한다.

## 산출물

- `metrics.json`: 검증 20개, 선택 모델과 주요 metric
- `split_manifest.json`, `internal_split.csv`: development 내부 분할
- `selection_manifest.json`: 모델 선택과 checkpoint hash
- `models/*.pkl`: 동결된 세 모델
- `model_results.csv`: 후보 9개 결과
- `validation_predictions.csv`: validation 예측
- `permutation_importance.csv`: feature importance 원수치
- `feature_exclusions.csv`: 제외 weld와 이유
- `INTERPRETATION.md`: 실행 결과 자동 해석
- `figures/*.png`: split, 회귀, 분류, residual, importance 그림 7개

## 실행

```bash
conda activate physicsnemo-study
python scripts/run_rsw_m6_1_external_ctq_baseline.py
python -m pytest tests/test_rsw_external_ctq_baseline.py -q
ruff check src/physicsnemo_study/training/rsw_external_ctq_baseline.py \
  src/physicsnemo_study/training/rsw_external_ctq_report.py \
  scripts/run_rsw_m6_1_external_ctq_baseline.py \
  tests/test_rsw_external_ctq_baseline.py
```

정상 결과는 `20/20 PASS`이면서 external test outcome 사용, force feature, physical-twin 보정,
RSW-SIM-V3가 모두 `false`인 상태다.
