# M5.2 동결 모델 test·OOD 평가

## 목적과 누수 방지

M5.1에서 사전에 고정한 규칙은 64-sample overfit NRMSE 5%를 통과한 후보 중 validation
Tmax MAE가 가장 작은 모델을 고르는 것이다. 네 variant가 모두 gate를 통과했고
`data_only`가 validation Tmax MAE `20.516 K`로 선택됐다.

M5.2는 M5.1 결과 JSON, 선택 checkpoint, M5/M5.1 설정 파일의 SHA-256을 검사한 후에만
test를 연다. test 결과는 보고용이며 이를 보고 모델이나 loss weight를 다시 선택하지 않는다.

## 지표

OOD 악화비는 같은 지표의 IID test 값에 대한 비율이다.

\[
D_{OOD}(m)=\frac{m_{OOD}}{m_{IID}}.
\]

`D>1`이면 OOD에서 오차 또는 물리 위반이 증가했다는 뜻이다. Tmax signed bias는

\[
b_{T_{max}}=\frac{1}{N}\sum_i(\widehat T_{max,i}-T_{max,i})
\]

이며 양수는 과대예측, 음수는 과소예측이다.

## 결과

| Split | Field NRMSE | Tmax MAE [K] | Tmax bias [K] | 과소예측률 | Residual RMSE |
|---|---:|---:|---:|---:|---:|
| test IID | 0.2120 | 48.112 | +48.112 | 0% | 0.3801 |
| process OOD | 0.5637 | 319.588 | -311.512 | 75% | 1.2260 |
| corner OOD | 0.4434 | 426.726 | -426.726 | 100% | 1.8190 |

- process OOD Tmax MAE는 IID의 `6.643`배다.
- corner OOD Tmax MAE는 IID의 `8.869`배다.
- process OOD reference Tmax 범위는 `581.48..1623.33 K`인데 예측은
  `547.99..796.74 K`에 머문다.
- corner OOD reference Tmax 범위는 `1276.68..1486.00 K`인데 예측은
  `873.51..1055.62 K`에 머문다.

고온 OOD에서 FNO가 출력 범위를 충분히 외삽하지 못한다. 특히 corner OOD의 모든 sample을
과소예측하므로 현재 surrogate를 공정 품질 판정에 사용하면 위험한 false-safe 방향의 오차가
생길 수 있다.

## 데이터 분포 해석

| Split | Samples | Mean electrical energy [J] | Mean reference Tmax [K] |
|---|---:|---:|---:|
| train | 64 | 45.34 | 552.08 |
| validation | 16 | 27.07 | 424.85 |
| test IID | 16 | 41.48 | 514.44 |
| process OOD | 16 | 94.92 | 946.99 |
| corner OOD | 8 | 122.44 | 1396.34 |

validation이 train과 IID test보다 저에너지 쪽에 치우쳐 있어 validation Tmax 선택이 고온
일반화를 대표하지 못했다. 이는 결정론성 문제가 아니라 smoke split의 작은 표본 수와 단순
무작위 범위 표본화 문제다.

## 실행과 산출물

```bash
conda activate physicsnemo-study
python scripts/run_rsw_m5_2.py
```

결과는 `artifacts/m5_2_frozen_test_evaluation/`에 JSON·CSV·해석문과 다음 그림으로 저장된다.

1. `sealed_split_accuracy.png`
2. `sealed_tmax_parity.png`
3. `sealed_physics_constraints.png`
4. `sealed_ood_degradation.png`
5. `sealed_worst_fields.png`

## 다음 결정

E05 checkpoint는 그대로 보존한다. 다음 실험은 기존 test에 맞춰 E05를 재조정하는 것이
아니라, 더 큰 space-filling simulation dataset과 새로운 validation/test seed를 사용하는
별도 데이터 버전으로 등록해야 한다. 동시에 Mendeley 실측 CTQ와 Polito 공장 시계열을 위한
M6 adapter를 구현한다.
