# M4 mean·3D CNN와 64-sample overfit

## 목적

M4는 PhysicsNeMo 모델 비교 전에 데이터 로더, 정규화, checkpoint 선택, 물리 단위 평가가
정상인지 확인하는 기준선 단계다. M3의 train 64개를 모두 사용하며 validation/test sample을
학습에 섞지 않는다.

## 모델

### Train mean baseline

정규화된 train target의 각 시공간 voxel 평균을 저장한다.

\[
\bar y(t,r,z)=\frac{1}{64}\sum_{i=1}^{64} y_i(t,r,z)
\]

새 입력이 들어와도 항상 같은 field를 출력한다. 따라서 CNN이 입력의 전류, 재료, 접촉조건을
실제로 활용하는지 확인하는 최소 비교선이다.

### Residual 3D CNN

입력과 출력 계약은 다음과 같다.

```text
input  [B, 10, T, R, Z]
output [B,  1, T, R, Z]
```

3D convolution은 시간·반경·축 방향의 국소 패턴을 함께 본다. residual block은

\[
h_{l+1}=\operatorname{GELU}(h_l+F_l(h_l))
\]

형태다. 이 모델은 전체 current schedule을 한 번에 받는 direct operator 기준선이며,
실시간 인과 모델로 해석하지 않는다.

## 학습과 평가 수식

학습 loss는 train-only 통계로 정규화한 temperature-rise field의 MSE다.

\[
\mathcal L_{\mathrm{MSE}}
=\frac{1}{N}\sum_j(\hat y_j-y_j)^2
\]

보고용 field NRMSE는 kelvin 단위 temperature rise로 복원한 뒤 계산한다.

\[
\operatorname{NRMSE}
=\sqrt{\frac{\sum_j(\Delta\hat T_j-\Delta T_j)^2}
{\sum_j(\Delta T_j)^2}}
\]

최대온도는 입력의 초기온도를 다시 더해 계산한다.

\[
\hat T_{\max}=\max_{t,r,z}\{T_0(t,r,z)+\Delta\hat T(t,r,z)\}
\]

현재 melt temperature가 근거와 함께 확정되지 않았으므로 melt diameter·boundary 지표는
`null`로 남긴다. M4의 임시 checkpoint 선택 지표는 validation Tmax MAE다.

## 두 checkpoint를 분리하는 이유

- `cnn3d/best.ckpt`: validation Tmax MAE가 가장 낮은 epoch다. 일반화 비교에 사용한다.
- `cnn3d/last.ckpt`: 마지막 epoch다. 64개 train sample을 충분히 맞출 수 있는지 검사한다.

한 checkpoint로 두 목적을 섞으면 validation에 좋은 시점과 train 암기가 가장 강한 시점이
다를 때 결과를 잘못 해석하게 된다.

64-sample gate는 마지막 checkpoint의 train field NRMSE가 5% 이하이고 mean보다 낮을 때
통과한다. 5%는 pipeline 구현 확인용 engineering gate이지 산업 품질 허용오차가 아니다.

## 실행

```bash
conda activate physicsnemo-study
python scripts/run_rsw_m4.py
```

기본 설정은 CUDA, fp32, 100 epoch, train 64개, validation 16개다. test split은 개발 중
열어보지 않는다. 모델과 선택 규칙을 동결한 뒤에만 다음처럼 평가 split을 명시한다.

```bash
python scripts/run_rsw_m4.py \
  --evaluation-splits validation test_iid test_process_ood test_corner
```

## 산출물

```text
artifacts/m4_baseline_overfit64/
├── mean/best.ckpt
├── cnn3d/best.ckpt
├── cnn3d/last.ckpt
├── m4_results.json
├── m4_results.csv
├── m4_overfit_results.csv
├── visualization_metadata.json
├── INTERPRETATION.md
└── figures/
    ├── cnn3d_training_history.png
    ├── tmax_parity.png
    ├── tmax_error_histogram.png
    └── cnn3d_worst_field_panel.png
```

그림의 worst sample은 사람이 고르지 않고 validation Tmax 절대오차가 가장 큰 index로
결정하며 stack/schedule/contact ID를 JSON에 함께 기록한다.

## 결과 확인 순서

1. `overfit_check.passed`가 `true`인지 본다.
2. `best.ckpt`의 selected epoch와 `last.ckpt`의 epoch가 다른지 확인한다.
3. validation에서 CNN의 field NRMSE와 Tmax MAE가 mean보다 낮은지 확인한다.
4. parity와 signed error histogram으로 과대·과소예측 방향을 본다.
5. worst field panel에서 오차가 reference peak 시점의 어느 공간에 집중되는지 본다.

overfit은 통과하지만 validation이 나쁘면 코드가 학습할 능력은 있으나 sample 수, 분포,
CNN inductive bias 또는 regularization이 일반화에 부족하다는 뜻이다. 이 경우 overfit
checkpoint를 성능 모델로 채택하지 않는다.
