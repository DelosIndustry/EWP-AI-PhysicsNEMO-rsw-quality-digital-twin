# M5.2 frozen test·OOD 평가 해석

## 결과

| Split | Field NRMSE | Tmax MAE [K] | Tmax bias [K] | Underpredict | Residual RMSE |
|---|---:|---:|---:|---:|---:|
| test_iid | 0.2120 | 48.112 | 48.112 | 0.0% | 0.3801 |
| test_process_ood | 0.5637 | 319.588 | -311.512 | 75.0% | 1.2260 |
| test_corner | 0.4434 | 426.726 | -426.726 | 100.0% | 1.8190 |

## OOD 해석

- process OOD / IID Tmax MAE 비: `6.643`
- corner OOD / IID Tmax MAE 비: `8.869`
- 1보다 크면 IID보다 오차가 증가했음을 뜻한다.
- signed bias가 양수면 과대예측, 음수면 과소예측이다. 특히 OOD의 큰 음수 bias는 고온 영역을 충분히 외삽하지 못했음을 뜻한다.

이 평가는 해시로 동결된 단일 checkpoint의 보고용 결과다. test 결과를 보고 모델이나 loss를 다시 선택하지 않는다. 표본 수가 IID 16, process OOD 16, corner OOD 8로 작으므로 불확실성이 크며 실측 용접 품질 성능으로 해석할 수 없다.
