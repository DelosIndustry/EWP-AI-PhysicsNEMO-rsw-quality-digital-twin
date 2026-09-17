# M4 mean·3D CNN 결과 해석

## 평가 계약

- mean은 64개 train target의 voxel별 평균이며 개별 입력을 보지 않는다.
- CNN checkpoint는 validation Tmax MAE로 선택하고 test split에는 맞추지 않는다.
- 64-sample overfit은 마지막 checkpoint를 train에서 별도로 검사한다.
- 현재 용융온도가 미설정이므로 melt 품질 지표는 아직 계산하지 않는다.

## validation 결과

- `mean`: field NRMSE=0.9010, Tmax MAE=114.851 K, negative rise=0.00%
- `cnn3d`: field NRMSE=0.3672, Tmax MAE=45.873 K, negative rise=8.91%

## 64-sample overfit gate

- CNN train field NRMSE=0.0235, 목표=0.0500, 통과=True

## 그림 읽는 법

1. `cnn3d_training_history.png`: train 감소와 validation 악화를 함께 본다.
2. `tmax_parity.png`: 대각선에서 멀수록 최대온도 오차가 크다.
3. `tmax_error_histogram.png`: 양수는 과대, 음수는 과소예측이다.
4. `cnn3d_worst_field_panel.png`: validation에서 Tmax 절대오차가 가장 큰 sample을 고정 규칙으로 선택했다.

## 해석 한계

- 8×8×8 smoke grid와 64개 합성 sample 결과는 산업 정확도를 뜻하지 않는다.
- 3D CNN은 전체 current schedule을 한 번에 보므로 실시간 인과 예측 모델이 아니다.
- threshold 미달이면 FNO/Transolver 전체 비교보다 baseline·학습 설정을 먼저 점검한다.
