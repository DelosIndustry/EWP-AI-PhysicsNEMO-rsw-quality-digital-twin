# M5.1 FNO 물리 제약

## 목적

M5에서 64-sample overfit gate를 통과한 FNO3D에 초기조건, 온도상승 비음수, M2 열수지
residual을 한 항씩 추가한다. test와 OOD split은 모델과 loss weight를 동결하기 전까지 열지
않는다.

## M2와 같은 이산 residual

각 cell `i`와 backward-Euler interval `n`에 대해 다음 power balance를 사용한다.

\[
R_i^n = \frac{C_i}{\Delta t_n}(\theta_i^n-\theta_i^{n-1})
+ \sum_j G_{ij}(\theta_i^n-\theta_j^n)
+ h_i A_i\theta_i^n-q_i^nV_i.
\]

여기서 `theta=T-T_initial`, `C=rho_cp V`, `G`는 M2 finite-volume face
conductance다. 중앙 판재 접촉면에는 thermal contact resistance를 포함하고 위·아래 전극
경계에는 Robin cooling을 포함한다. 학습에는 power residual을 그대로 쓰지 않고

\[
\widehat R_i^n = \frac{\Delta t_n}{C_i\,s_\theta}R_i^n
\]

으로 무차원화한다. `s_theta`는 train temperature-rise 표준편차다. 따라서
`widehat R=1`은 한 time step에서 표준편차 한 배의 등가 온도 오차라는 뜻이다.

## Loss ablation

\[
L = L_{data}+\lambda_{IC}L_{IC}
+\lambda_{neg}L_{neg}+\lambda_{R}L_R.
\]

- `L_data`: 정규화된 temperature-rise MSE
- `L_IC`: 최초 시각의 `theta=0` 제약
- `L_neg`: `ReLU(-theta)` 제곱 평균
- `L_R`: 무차원 M2 residual 제곱 평균

`m5_1_physics.yaml`은 data-only → initial → nonnegative → residual 순서의 누적
one-factor-at-a-time 비교를 고정한다.

## 먼저 실행할 reference gate

```bash
conda activate physicsnemo-study
python scripts/audit_rsw_m5_1_reference_residual.py
```

정답장 최악의 등가 온도 residual이 `1e-3 K` 이하이고 초기조건 오차가 `1e-6 K` 이하일 때만
physics-constrained FNO 학습으로 넘어간다. 결과 JSON, PNG 세 장, 한국어 해석문은
`artifacts/m5_1_reference_residual/`에 저장된다.

현재 동결 M3 데이터 결과는 최악 residual `3.510684e-4 K`, 초기조건 오차 `0 K`로
reference gate를 통과했다.

## FNO ablation 실행

전체 비교는 다음 명령으로 실행한다.

```bash
conda activate physicsnemo-study
python scripts/run_rsw_m5_1.py
```

빠른 경로 확인만 다시 수행할 때는 다음처럼 일부 variant와 epoch를 지정한다.

```bash
python scripts/run_rsw_m5_1.py \
  --variants data_only plus_thermal_residual \
  --epochs 1 \
  --output-dir projects/rsw_quality_digital_twin/artifacts/m5_1_path_check
```

전체 결과에는 validation accuracy와 세 물리 위반 지표, 64-sample train overfit 결과가
포함된다. 각 variant는 같은 seed, 초기화, 데이터 순서, optimizer와 epoch budget을 사용한다.
100 epoch 후 overfit NRMSE 5% 이하인 variant만 최종 선택 후보가 된다.

```text
artifacts/m5_1_physics_ablation/
├── {variant}/{best,last}.ckpt
├── m5_1_results.json
├── m5_1_results.csv
├── m5_1_overfit_results.csv
└── figures/
    ├── physics_ablation_training.png
    ├── physics_ablation_accuracy.png
    ├── physics_ablation_constraints.png
    └── physics_ablation_worst_fields.png
```

이 단계에서도 evaluation split은 validation으로 고정되며 test IID·process OOD·corner OOD는
