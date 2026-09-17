# 실험 계획

## 공통 규칙

- solver 검증 전에는 ML 성능 실험을 시작하지 않는다.
- 모델 비교에서는 split, 정규화, loss, epoch budget과 checkpoint metric을 고정한다.
- 한 ablation에서 독립변수 하나만 바꾼다.
- 최소 3개 seed의 평균·표준편차와 worst seed를 보고한다.
- simulation IID/OOD와 experimental 결과를 별도 표로 보고한다.
- validation에서 선택한 모델·threshold를 test 결과에 맞춰 바꾸지 않는다.

## 단계별 실험

| ID | 질문 | 비교 | 주요 지표 |
|---|---|---|---|
| E00 | 후보 모델 계약이 맞는가? | FNO3D / Transolver random tensor | shape, finite gradient, memory |
| E01 | 전기 solver가 맞는가? | 1D 해석해 / 2D grid | potential error, current conservation |
| E02 | 열 연성이 맞는가? | 무발열 / 균일발열 / grid-time refinement | energy error, convergence rate |
| E03 | Dataset이 재현되는가? | same/different seed | file hash, manifest, group leakage |
| E04 | 단순 모델보다 나은가? | mean / MLP-CTQ / 3D CNN | field·CTQ error, latency |
| E05 | 어느 PhysicsNeMo 모델이 맞는가? | FNO3D / Transolver | melt diameter MAE, nRMSE, memory |
| E06 | 시간 표현이 중요한가? | full trajectory / final step / autoregressive | rollout drift, CTQ error |
| E07 | physics loss가 돕는가? | weight 0 / 0.01 / 0.1 | PDE residual, OOD CTQ error |
| E08 | 공개 실험과 연결되는가? | simulation proxy / measured nugget | calibration/test MAE, sim-real gap |
| E09 | 안전한 자동판정인가? | point / conformal / OOD gate | false escape, review rate, coverage |
| E10 | mesh 모델이 필요한가? | selected grid model / MeshGraphNet | geometry OOD, cost, accuracy |

## 최소 결과 schema

```text
experiment_id
model
seed
simulation_manifest_id
external_manifest_id
split
field_nrmse
tmax_mae_k
melt_boundary_dice
melt_diameter_proxy_mae_mm
relative_energy_balance_error
false_escape_rate
false_reject_rate
review_rate
ood_recall
p50_latency_ms
p95_latency_ms
parameter_count
peak_gpu_memory_mb
```

적용되지 않는 값은 0으로 채우지 않고 `NaN/not applicable`로 기록한다.

## 모델 선택 순서

1. 수치 check와 OOD gate가 실패한 모델은 제외한다.
2. validation melt diameter proxy MAE를 우선한다.
3. false escape가 더 낮은 모델을 우선한다.
4. field nRMSE와 worst 5% error를 확인한다.
5. 정확도가 실질적으로 같으면 latency와 memory가 작은 모델을 선택한다.

## 중단 규칙

- potential 해석해 또는 current conservation test 실패 시 E02로 가지 않는다.
- energy balance나 grid/time 수렴 실패 시 데이터를 생성하지 않는다.
- 64-sample overfit 실패 시 전체 학습을 시작하지 않는다.
- 동일 group이 train/test에 겹치면 해당 manifest를 폐기한다.
- external test로 접촉계수나 threshold를 다시 맞추면 새 실험과 manifest를 만든다.
- Transolver가 memory budget을 넘으면 token subsampling을 즉시 적용하지 않고 먼저 실패
  원인과 공정한 계산예산을 기록한다.

## 최종 그림

1. geometry/material/contact/current 입력과 `q_joule`, reference/predicted `T`, error
2. 시간에 따른 중심부 온도와 전류 schedule
3. reference/predicted melt boundary와 diameter scatter
4. FNO3D/Transolver/CNN 정확도-지연시간-memory 비교
5. IID/process OOD CTQ error distribution
6. conformal interval에 따른 false escape-review rate trade-off
7. simulation proxy와 experimental nugget 사이의 sim-to-real gap
