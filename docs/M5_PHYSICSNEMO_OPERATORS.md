# M5 PhysicsNeMo FNO3D·Transolver 비교

## 목적

M4의 mean·3D CNN 기준선을 고정한 뒤 동일한 M3 manifest와 계산 규칙으로 PhysicsNeMo
FNO3D와 Transolver를 비교한다. 현재 단계는 simulation field 학습 능력을 보는 것이며 실제
점용접 품질을 검증하는 단계가 아니다.

## 모델 표현 차이

### FNO3D

FNO는 `[B,C,T,R,Z]` regular grid를 직접 받는다. Fourier layer의 핵심은 저주파 mode에서
학습한 spectral kernel을 적용하는 것이다.

\[
v_{l+1}(x)=\sigma\left(Wv_l(x)+
\mathcal F^{-1}\left(R_\theta\,\mathcal F(v_l)\right)(x)\right)
\]

여기서 세 operator 축은 세 공간축이 아니라 `(time, radial, axial)`이다. 현재 8×8×8
smoke grid에서는 축마다 Fourier mode 6개를 사용한다.

### Transolver

Transolver는 같은 field를 512개 point token으로 바꾼다.

```text
[B, 10, 8, 8, 8] → [B, 512, 10]
coordinates        → [B, 512, 3]  # normalized t,r,z
```

PhysicsNeMo 2.1.1의 3D `structured_shape` 경로 대신 명시적인 point 좌표를 전달하고, 출력
token을 다시 `[B,1,T,R,Z]`로 복원한다.

## 공정한 비교 계약

- simulation manifest: `8a39a964...ae8be`
- train/validation: 64/16 samples
- seed: 42
- optimizer: AdamW, learning rate `1e-3`, weight decay 0
- loss: normalized temperature-rise field MSE
- epoch budget: 100
- best checkpoint: minimum validation Tmax MAE
- last checkpoint: 64-sample overfit 진단
- test/OOD split: 아직 접근하지 않음

모델별 batch 크기나 epoch를 바꾸지 않았다. checkpoint는 현재 resume을 지원하지 않으므로
디스크 사용량을 줄이기 위해 model state와 provenance만 저장한다.

## 실행

```bash
conda activate physicsnemo-study
python scripts/run_rsw_m5.py
```

결과는 다음 위치에 생성된다.

```text
artifacts/m5_operator_comparison/
├── fno3d/{best,last}.ckpt
├── transolver/{best,last}.ckpt
├── m5_results.json
├── m5_results.csv
├── m5_overfit_results.csv
├── visualization_metadata.json
├── INTERPRETATION.md
└── figures/*.png
```

## 100-epoch 결과

### 64-sample overfit gate

| 모델 | Train field NRMSE | 기준 | 판정 |
|---|---:|---:|---|
| FNO3D | 4.71% | ≤5% | PASS |
| Transolver | 9.34% | ≤5% | FAIL |

FNO는 구현·optimizer 경로가 작은 데이터 자체를 학습할 수 있다는 gate를 통과했다.
Transolver는 200-epoch 추가 진단에서도 7.13%여서 단순히 100 epoch가 조금 부족한 정도로
단정할 수 없다. hidden size, slice 수, learning-rate schedule의 독립적인 재검토가 필요하다.

### Validation best checkpoint

| 모델 | 선택 epoch | Field NRMSE | Tmax MAE | 음의 상승률 | p95 latency |
|---|---:|---:|---:|---:|---:|
| M4 CNN | 20 | 36.72% | 45.87 K | 8.91% | 0.61 ms |
| FNO3D | 20 | 31.09% | 20.52 K | 15.38% | 3.64 ms |
| Transolver | 23 | 29.38% | 12.46 K | 4.79% | 2.98 ms |

두 PhysicsNeMo 모델 모두 CNN보다 validation 정확도가 높다. Transolver는 validation 지표,
파라미터 수와 지연시간에서 FNO보다 좋지만 overfit gate를 통과하지 못했다.

## 선택 해석

현재 자동 규칙은 overfit gate를 통과한 모델만 다음 실험 후보로 인정한다. 따라서 formal
candidate는 FNO3D다. 이것은 FNO가 모든 면에서 우월하다는 뜻이 아니다.

- FNO3D: gate PASS, 하지만 validation 음의 temperature rise가 15.38%다.
- Transolver: validation은 가장 좋지만 작은 train set을 충분히 맞추지 못했다.
- melt threshold가 없으므로 선택 metric도 임시 Tmax MAE다.

따라서 다음 M5.1에서는 FNO3D에 thermal residual 또는 초기조건 제약을 추가해 음의
temperature rise와 OOD 물리 위반이 감소하는지 확인한다. Transolver는 공정한
hyperparameter ablation을 별도 실험 ID로 남기기 전까지 탈락 상태를 유지한다.

## 그림 읽는 순서

1. `operator_training_histories.png`: train 수렴과 validation 최적 시점 차이를 본다.
2. `validation_accuracy_comparison.png`: mean·CNN 대비 accuracy 개선을 본다.
3. `latency_accuracy_pareto.png`: 개선된 accuracy의 inference cost를 본다.
4. `operator_tmax_parity.png`: 온도 범위별 편향을 본다.
5. `operator_tmax_error_histogram.png`: 과대·과소예측 방향을 본다.
6. `operator_worst_field_panels.png`: 각 모델의 결정론적 worst sample 공간 오차를 본다.

## 주의

- validation sample이 16개뿐이므로 모델 순위를 일반화하면 안 된다.
- test 결과에 맞춰 gate나 hyperparameter를 변경하지 않는다.
- 실제 nugget 직경과 강도 검증 전에는 생산 품질 개선 효과를 주장하지 않는다.
