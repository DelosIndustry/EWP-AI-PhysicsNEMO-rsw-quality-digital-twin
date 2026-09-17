# M6.2 동결 외부 test 평가

## 목적

M6.1에서 선택하고 SHA-256으로 동결한 모델 세 개를 변경하지 않은 채, 분리해 둔 Mendeley
external test 99 weld를 최종 보고용으로 평가한다. 이 단계는 모델을 개선하거나 다시 고르는
단계가 아니다.

- model fit: 금지
- model/hyperparameter 재선택: 금지
- threshold 조정: 금지
- external test 결과에 따른 기존 manifest 수정: 금지
- 자동 품질 합격 및 physical-twin 주장: 금지

평가 전에 config, evaluator, runner, selection manifest와 모델 파일의 hash를 검증한다. 결과는
임시 디렉터리에서 완성한 뒤 한 번에 게시하며, 이미 결과 디렉터리가 있으면 덮어쓰지 않는다.

## 평가 지표

회귀 오차는 다음과 같이 계산한다.

\[
\mathrm{MAE}=\frac{1}{n}\sum_{i=1}^{n}|\hat y_i-y_i|,
\qquad
\mathrm{RMSE}=\sqrt{\frac{1}{n}\sum_{i=1}^{n}(\hat y_i-y_i)^2}
\]

\[
R^2=1-\frac{\sum_i(y_i-\hat y_i)^2}{\sum_i(y_i-\bar y)^2}
\]

분류에서는 accuracy만 보지 않고 class별 recall, balanced accuracy와 macro-F1을 함께 본다.

\[
\mathrm{Recall}_c=\frac{TP_c}{TP_c+FN_c},
\qquad
\mathrm{BalancedAccuracy}=\frac{1}{C}\sum_{c=1}^{C}\mathrm{Recall}_c
\]

\[
\mathrm{MacroF1}=\frac{1}{C}\sum_{c=1}^{C}F1_c
\]

## 표본과 제외 규칙

- 원본 test: 99 weld, 808 record
- feature-eligible: 98 weld
- sample 250: communication error로 current가 `-99` 하나뿐이어서 feature 계산 불가
- nugget/category 평가: 98 weld
- pull-test 평가: 97 weld
- sample 185: 서로 다른 pull-test 값 두 개가 있어 scalar target을 null로 보존했으므로 제외
- 0.60–0.66 mm 목표 stack: category/nugget 93 weld, pull-test 92 weld

force는 입력 feature에 포함하지 않았으며, development outcome도 이 실행에서 읽지 않았다.

## 동결 외부 test 결과

### 회귀

| CTQ | Test MAE | 95% bootstrap CI | Test RMSE | Test R² | Validation→test |
|---|---:|---:|---:|---:|---:|
| Nugget diameter | 0.2368 mm | [0.1944, 0.2906] | 0.3367 mm | 0.2055 | MAE 1.019배 |
| Pull test | 147.62 N | [112.52, 187.98] | 243.57 N | 0.5830 | MAE 0.848배 |

Nugget MAE는 validation과 거의 같지만 R²의 95% bootstrap 구간은 `[-0.0693, 0.4490]`으로
0을 포함한다. 평균 절대오차만 보면 안정적으로 보일 수 있으나 weld 간 직경 차이를 설명하는
능력은 약하고 불확실하다. 산점도에서는 작은 값은 높게, 큰 값은 낮게 예측하는 평균 회귀가
보인다. Pull-test도 극단값에서 같은 압축 현상이 있으며 R² 구간은 `[0.1886, 0.7704]`다.

### 품질 category

| 지표 | Test | 95% class-stratified bootstrap CI | Validation |
|---|---:|---:|---:|
| Accuracy | 0.8878 | [0.8469, 0.9286] | 0.9231 |
| Balanced accuracy | 0.5682 | [0.4015, 0.6629] | 0.6175 |
| Macro-F1 | 0.6003 | [0.4479, 0.6538] | 0.6813 |

실제/예측 순서를 Bad, Explode, Good으로 고정한 confusion matrix는 다음과 같다.

\[
\begin{bmatrix}
3 & 1 & 0\\
0 & 0 & 6\\
0 & 4 & 84
\end{bmatrix}
\]

- Bad recall: 3/4 = 0.75
- Explode recall: 0/6 = 0.00
- Good recall: 84/88 = 0.9545

Explode 6개를 모두 Good으로 예측했다. 만약 Good을 자동 합격으로 바로 연결하면 이 여섯 건은
모두 잠재 false escape가 된다. 따라서 accuracy 0.8878은 생산 품질 gate 승인 근거가 아니다.

## Bootstrap 해석

회귀는 weld 1개를 독립 단위로 5,000회 ordinary bootstrap했다. 분류는 각 반복에서
Bad/Explode/Good의 원래 표본수를 유지하는 class-stratified weld bootstrap을 사용했다.

\[
CI_{95\%}=\left[Q_{0.025}(m^{*(b)}),\ Q_{0.975}(m^{*(b)})\right]
\]

Explode는 관측된 6개가 모두 오분류되어 비모수 bootstrap에서도 모든 반복의 recall이 0이므로
구간이 `[0, 0]`으로 퇴화한다. 이것은 모집단 recall을 오차 없이 0으로 안다는 뜻이 아니다.
표본이 너무 작고 성공 사례가 전혀 없다는 뜻이며, 다음 독립 평가에서는 exact binomial 또는
Wilson interval을 보조 구간으로 함께 사용해야 한다.

## 현재 부족한 부분

### 데이터

1. test가 한 연구실의 99 weld뿐이고 Bad 4개, Explode 6개로 매우 불균형하다.
2. 다른 용접기, 판재 조합, 전극 마모 상태, 공장 및 시간대에 대한 독립 검증이 없다.
3. pressure는 electrode tip force 실측값이 아니라 공압 설정값이다.
4. 전압·동저항과 신뢰 가능한 시간축이 없어 Joule energy를 실측에서 복원할 수 없다.
5. 내부 온도장 `T(r,z,t)` 측정값이 없어 PhysicsNeMo field 예측을 검증할 수 없다.

### 모델

1. 현재 CTQ 모델은 Random Forest이며 PhysicsNeMo operator가 아니다.
2. 극단적인 nugget/pull 값이 평균 쪽으로 압축된다.
3. 입력 범위 밖 extrapolation과 OOD를 판정하는 검증된 detector가 없다.
4. single holdout 결과이며 nested CV, cross-site validation과 probability calibration이 없다.
5. 목표 stack category에는 Bad 1개와 Explode 6개뿐이고 두 class recall이 모두 0이다.

### 생산 품질 의사결정

1. false escape 비용과 허용 한계가 공정 요구사항으로 확정되지 않았다.
2. PASS/REVIEW/FAIL threshold가 아직 calibration되지 않았다.
3. 불확실성 구간은 population 성능의 표본 불확실성이지 개별 weld 예측 불확실성이 아니다.
4. Mendeley test는 이미 최종 평가에 사용했으므로 이후 모델 개선에 다시 사용할 수 없다.

## 산출물과 시각화

- `metrics.json`: 27개 gate와 전체 요약
- `evaluation_receipt.json`: selection/test source/output hash 및 실행 금지사항
- `test_metrics.csv`: 동결 모델의 test 지표
- `test_predictions.csv`: weld별 측정값·예측값·residual/probability
- `bootstrap_intervals.csv`: 5,000회 bootstrap 구간
- `target_stack_metrics.csv`: 0.60–0.66 mm subgroup 결과
- `feature_exclusions.csv`: 제외 sample과 이유
- `INTERPRETATION.md`: 실행 시 자동 생성된 해석
- `figures/*.png`: 구성, 산점도, residual, confusion matrix, validation 비교, bootstrap,
  class recall 그림 7개

## 실행 상태

최종 evaluation receipt ID는
`4dbee4092cde82bc3735a0f055fde8ff7a041dd15cf7efd237de27cd29d5362b`다.
결과 디렉터리가 이미 존재하므로 runner를 다시 실행하면 평가 전에 중단되는 것이 정상이다.

```bash
conda activate physicsnemo-study
python -m pytest tests/test_rsw_external_ctq_frozen_test.py -q
ruff check src/physicsnemo_study/training/rsw_external_ctq_frozen_test.py \
  scripts/run_rsw_m6_2_frozen_external_test.py \
  tests/test_rsw_external_ctq_frozen_test.py
```

## 다음 단계

M7에서는 이 test로 threshold를 맞추지 않는다. development 데이터만으로 OOD와 REVIEW 정책을
설계하고, Polito fault 데이터는 서로 다른 label 의미를 유지한 채 설비 이상 감지 보조 평가에
사용한다. 개선 모델을 최종 승인하려면 Mendeley test와 독립된 새 holdout이 필요하다.
