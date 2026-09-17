# M7.1 개발 데이터 전용 품질 게이트와 OOD feasibility

## 목적과 결론

M7.1은 Mendeley **development weld 393개만** 사용해 측정 category를
`PASS / REVIEW / FAIL` 의사결정으로 바꾸는 방법을 실험한다. M6.2에서 한 번 사용한 external
test 99개는 모델 학습, 확률 보정, 임계값 선택, OOD 규칙 선택에 사용하지 않았다.

결론은 명확하다. 보수적인 게이트는 개발 교차예측에서 false escape를 `1/42 = 2.38%`로
낮췄지만, `359/393 = 91.35%`를 REVIEW로 보냈다. 따라서 **안전 경고 feasibility는
확인했지만 생산 자동화 게이트는 아니다.**

## 왜 nested cross-fitting을 사용했는가

같은 weld로 모델을 학습하고 임계값까지 정한 뒤 그 weld의 성능을 재면 결과가 낙관적으로
부풀 수 있다. 이를 막기 위해 두 층의 교차검증을 사용했다.

1. 바깥 5-fold: 각 weld의 최종 교차예측을 만드는 평가 분리
2. 안쪽 4-fold: 바깥 train 안에서만 raw 확률을 만들고 확률 보정기와 임계값을 설정
3. 바깥 holdout: 모델·보정기·임계값을 모두 고정한 뒤 한 번만 판정

각 `sample_id`는 바깥 holdout에 정확히 한 번만 등장했다. Random Forest 계열과
hyperparameter는 M6.1에서 선택된 `random_forest_balanced`를 그대로 사용했다.

## 확률 보정

세 category의 one-hot 정답을 \(y_{ic}\), 예측 확률을 \(p_{ic}\)라고 하면 multiclass
Brier score는 다음과 같다.

\[
\mathrm{Brier}_{multi}=\frac{1}{N}\sum_{i=1}^{N}\sum_c(p_{ic}-y_{ic})^2
\]

Good이 아닌 것을 failure로 묶으면 \(p_{fail}=1-p_{Good}\)이고 binary failure Brier
score는 다음과 같다.

\[
\mathrm{Brier}_{fail}=\frac{1}{N}\sum_{i=1}^{N}(p_{fail,i}-y_{fail,i})^2
\]

안쪽 OOF 확률 위에 multinomial logistic stacking을 적합했다. 바깥 교차예측 결과는 다음과
같다.

| 지표 | Raw RF | 보정 후 |
|---|---:|---:|
| Failure Brier | 0.07336 | 0.06039 |
| Failure ECE, 10 bins | 0.07676 | 0.02700 |
| Multiclass Brier | 0.15135 | 0.12525 |
| Multiclass log loss | 0.46552 | 0.26037 |
| 3-class balanced accuracy | 0.66406 | 0.60594 |
| 3-class macro-F1 | 0.69835 | 0.62067 |

보정은 확률 오차를 줄였지만 세 클래스의 argmax 구분 성능은 낮췄다. 특히 Explode recall은
raw `5/25 = 0.20`에서 보정 후 `0/25 = 0`이 됐다. 이는 calibration과 discrimination이
서로 다른 목표라는 증거다. M7.1 gate는 Explode를 별도 class로 자동 판정하기보다 높은
불확실성을 REVIEW로 보내는 데 의미가 있다.

## PASS / REVIEW / FAIL 임계값

안쪽 OOF에서 관측된 불량을 PASS로 보내지 않는 최소 PASS 임계값과, 관측된 Good을 FAIL로
보내지 않는 최소 FAIL 임계값을 fold마다 계산했다.

\[
\tau_{pass}=\operatorname{nextafter}\left(
\max_{i:y_i\ne Good}p_{Good,i},+\infty\right)
\]

\[
\tau_{fail}=\operatorname{nextafter}\left(
\max_{i:y_i=Good}(1-p_{Good,i}),+\infty\right)
\]

\[
D_i=\begin{cases}
PASS & p_{Good,i}\ge\tau_{pass}\ \land\ p_{fail,i}<\tau_{fail}\\
FAIL & p_{fail,i}\ge\tau_{fail}\ \land\ p_{Good,i}<\tau_{pass}\\
REVIEW & \text{그 외}
\end{cases}
\]

PASS 임계값은 fold별 `0.9597–0.9684`, FAIL 임계값은 `0.4343–0.6484`였다. PASS에 매우
높은 확신을 요구한 결과다. 안쪽 자료의 관측 오류 0은 미래 오류 0을 보장하지 않는다.

## OOD 후보

각 바깥 fold의 train만 기준으로 두 종류의 support 경고를 계산했다.

1. Training range: 한 feature라도 train의 열별 최솟값과 최댓값을 벗어나면 경고
2. Standardized Mahalanobis: 표준화 후 Ledoit–Wolf 공분산으로 다변량 거리를 계산하고
   train 거리의 97.5 percentile을 넘으면 경고

\[
d(x)=\sqrt{(z-\mu)^T\hat\Sigma_{LW}^{-1}(z-\mu)}
\]

Range는 10개(2.54%), Mahalanobis는 12개(3.05%), 합집합은 18개(4.58%)를 표시했다.
합집합 경고율은 Bad 35.29%, Explode 12.00%, Good 2.56%였다. 흥미로운 연관이지만 OOD
정답 label이 없으므로 검출 성능이나 인과관계로 해석할 수 없다.

## 정책별 개발 교차예측 결과

| 정책 | False escape | False reject | Review | Auto coverage |
|---|---:|---:|---:|---:|
| Argmax, review 없음 | 66.67% | 0.57% | 0.00% | 100.00% |
| 확률 abstention | 2.38% | 0.28% | 89.82% | 10.18% |
| 확률 + range | 2.38% | 0.28% | 90.59% | 9.41% |
| 확률 + Mahalanobis | 2.38% | 0.00% | 90.84% | 9.16% |
| 결합 진단 정책 | 2.38% | 0.00% | 91.35% | 8.65% |

결합 진단 정책의 category별 판정은 다음과 같다.

| 실제 category | PASS | REVIEW | FAIL |
|---|---:|---:|---:|
| Bad (17) | 0 | 8 | 9 |
| Explode (25) | 1 | 24 | 0 |
| Good (351) | 24 | 327 | 0 |

결합 정책 false escape의 95% class-stratified bootstrap 구간은 `[0, 7.14%]`, review rate는
`[88.80%, 93.89%]`, auto coverage는 `[6.11%, 11.20%]`다. Explode는 25개뿐이라
`1/25`의 변화도 4%p이며 불확실성이 크다.

## 산출물 읽기

- `metrics.json`: 전체 계약, raw/보정 지표, 정책, OOD, bootstrap 결과
- `cross_fitted_predictions.csv`: weld별 바깥 fold 확률·임계값·OOD·최종 판정
- `fold_thresholds.csv`: fold별 안쪽 calibration 임계값
- `policy_metrics.csv`: 다섯 정책의 trade-off
- `bootstrap_intervals.csv`: 결합 진단 정책의 95% 구간
- `01`–`07` PNG: 확률 분포, reliability, 판정, trade-off, OOD, confusion, 임계값
- `INTERPRETATION.md`: 실행 결과에서 자동 생성된 쉬운 해석
- `artifact_manifest.json`: 설정·구현·산출물 SHA-256

```bash
conda activate physicsnemo-study
python scripts/run_rsw_m7_1_development_gate.py
python -m pytest tests/test_rsw_quality_gate_feasibility.py -q
```

## 부족한 부분과 다음 gate

1. 현재 비용 `false escape:reject:review = 20:5:1`은 설명용이며 실제 공정 요구가 아니다.
2. OOD 정답이 없어 OOD recall 및 detector 선택을 하지 않았다.
3. 개발자료 자체가 한 연구실, 한 재질 중심이고 Bad/Explode가 42개뿐이다.
4. Mendeley external test는 이미 소비됐으므로 M7.1 정책을 그 test에 맞춰 재조정하지 않는다.
5. Polito normalized signal과 Fault label은 M7.2에서 별도 설비 이상 track으로 다룬다.
6. 생산 승인에는 시간·설비·stack을 분리한 새 독립 holdout과 사전에 합의한 품질 비용이 필요하다.
7. 내부 온도장 실측이 없으므로 PhysicsNeMo field 모델 검증과 physical twin 주장은 계속 차단한다.
