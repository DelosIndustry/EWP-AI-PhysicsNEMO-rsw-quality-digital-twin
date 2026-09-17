# M5 PhysicsNeMo operator 비교 해석

## 평가 계약

- FNO3D와 Transolver는 같은 64개 train, optimizer, epoch budget을 사용한다.
- best checkpoint는 validation Tmax MAE, overfit gate는 last checkpoint로 계산한다.
- test와 OOD split은 모델·규칙 동결 전까지 열지 않는다.
- melt CTQ가 미설정이므로 현재 선택은 Tmax MAE 기반의 임시 선택이다.

## 64-sample overfit gate

- `fno3d`: train field NRMSE=0.0471, 목표=0.0500, 통과=True
- `transolver`: train field NRMSE=0.0934, 목표=0.0500, 통과=False

## validation 결과

- `fno3d`: field NRMSE=0.3109, Tmax MAE=20.516 K, negative rise=15.38%, p95=3.638 ms
- `transolver`: field NRMSE=0.2938, Tmax MAE=12.457 K, negative rise=4.79%, p95=2.980 ms

## 임시 선택

- 선택 모델: `fno3d`
- eligible 모델: `['fno3d']`
- 다음 ablation은 선택 모델에만 thermal residual을 추가한다.

## 그림 읽는 법

1. `operator_training_histories.png`: 같은 예산에서 수렴과 과적합을 비교한다.
2. `validation_accuracy_comparison.png`: M4 기준선보다 실제로 나은지 본다.
3. `latency_accuracy_pareto.png`: 정확도 개선이 추론비용을 보상하는지 본다.
4. parity와 error histogram에서 과대·과소예측 방향을 본다.
5. worst field panel은 모델별 동일한 결정 규칙으로 실패 위치를 보여준다.

## 한계

- 결과는 8×8×8 합성 smoke grid에 한정되며 실제 용접 품질을 의미하지 않는다.
- PhysicsNeMo 사용은 확인되지만 정확도가 보장되는 것은 아니다.
- validation 선택 뒤 physics residual을 검증하고 나서 test를 한 번만 평가한다.
