# PhysicsNeMo 모델 검토와 결정

검토일: 2026-09-03

## 결론

FNO를 미리 승자로 정하지 않는다. 첫 공정한 비교는 다음 두 모델이다.

1. **PhysicsNeMo FNO3D**: `(time, r, z)` 고정 격자를 한 번에 예측
2. **PhysicsNeMo Transolver**: 같은 격자를 `(t,r,z)` 좌표를 가진 point token으로 변환해 예측

MeshGraphNet은 비정형 mesh가 준비되는 M8 후보이며, PINO/PhysicsInformer는 선택된
모델에 물리 residual을 추가하는 M5 학습 전략이다.

## 후보 비교

| 후보 | 적합한 표현 | 장점 | 현재 위험 | 프로젝트 역할 |
|---|---|---|---|---|
| FNO3D | 고정 `T x R x Z` 격자 | 전역 열확산과 전체 trajectory를 한 번에 처리 | 고주파 용융 경계, 격자 고정 | M4 공동 주후보 |
| Transolver | structured 또는 point/mesh token | 좌표·국소 상태 기반, 구조/비구조 자료 모두 가능 | memory와 attention 비용, 버전별 3D API 차이 | M4 공동 주후보 |
| MeshGraphNet | FEM node/edge graph | 불규칙 mesh와 재료·계면 topology 표현 | graph 생성, rollout 누적오차 | M8 mesh 확장 |
| PINO/PhysicsInformer | grid/mesh의 PDE residual | 적은 데이터에서 물리 regularization 가능 | residual scaling과 접촉 불연속 | M5 ablation |
| DeepONet | branch 입력 + 좌표 trunk | 임의 좌표·시간 query에 자연스러움 | 2.1.1에 범용 단일 builder가 없어 custom 조합 필요 | 탐색 후보 |
| PINN | 좌표 -> field | field label이 적은 경우와 역문제 | 강한 다중물리·불연속 접촉에서 학습 난도 큼 | 계수 역추정 후보 |
| DoMINO | 3D geometry point cloud | surface/volume field와 큰 형상 문제 | 점용접 solid electro-thermal용 turnkey가 아님 | 3D 장기 후보 |
| GeoTransolver | geometry-aware token | 형상 조건부 예측 강화 | 현재 2.1.1 환경에 module 없음 | 업그레이드 후 재검토 |

PhysicsNeMo 공식 FNO는 1D부터 4D field를 지원하므로 시공간 3D mapping을 직접 표현할 수
있다. [FNO API](https://docs.nvidia.com/physicsnemo/latest/physicsnemo/api/models/fnos.html)

Transolver는 physics-attention으로 structured와 unstructured mesh를 처리한다.
[Transolver API](https://docs.nvidia.com/physicsnemo/latest/physicsnemo/api/models/transolver.html)

MeshGraphNet은 simulation mesh의 vertex를 node, mesh connectivity를 edge로 바꾸므로 향후
전극과 판재의 비정형 FEM mesh에 적합하다.
[MeshGraphNet tutorial](https://docs.nvidia.com/physicsnemo/latest/user-guide/model_architecture/meshgraphnet.html)

PhysicsInformer는 autodiff, finite difference, meshless finite difference, spectral,
least-squares derivative로 PDE residual을 계산할 수 있다.
[PhysicsNeMo PINN guide](https://docs.nvidia.com/physicsnemo/latest/user-guide/pinns-tutorials/index.html)

DoMINO에는 transient conjugate heat-transfer 예제가 있지만, 점용접의 전기전도·접촉
조건은 직접 설계해야 하므로 첫 모델로 선택하지 않는다.
[Transient DoMINO CHT example](https://docs.nvidia.com/physicsnemo/latest/physicsnemo/examples/cfd/transient_conjugate_heat_transfer_tank_fill/README.html)

## 현재 환경에서 직접 확인한 사항

고정 환경은 `nvidia-physicsnemo==2.1.1`이다.

| module | import | 작은 forward/backward | 판정 |
|---|---:|---:|---|
| FNO | 성공 | `1 x 8 x 4 x 8 x 8 -> 1 x 1 x 4 x 8 x 8` 성공 | 사용 가능 |
| Transolver | 성공 | 256 point, 3좌표 입력에서 성공 | 사용 가능 |
| MeshGraphNet | 성공 | 이번 단계 미실행 | dependency 준비됨 |
| DoMINO | 성공 | 이번 단계 미실행 | 장기 후보 |
| `physicsnemo.sym` | 성공 | 이번 단계 미실행 | M5 후보 |
| GeoTransolver | 실패 | 해당 없음 | 2.1.1에서 사용하지 않음 |

최신 Transolver 문서는 3D `structured_shape`을 설명하지만, 2.1.1에서
`structured_shape=(T,R,Z)`와 `unified_pos=True`를 주면 다음 오류가 발생했다.

```text
ValueError: too many values to unpack (expected 2)
```

따라서 M0~M4에서는 `structured_shape=None`, `embedding_dim=3`으로 두고 `(t,r,z)` 좌표를
명시적으로 전달하는 검증된 point-token 경로를 사용한다. 버전 업그레이드는 별도 실험으로
취급하며 기준선 도중에 수행하지 않는다.

## 선택 gate

두 주후보는 같은 parameter budget을 억지로 맞추지 않고 다음을 모두 보고한다.

- parameter count와 peak GPU memory
- batch-1 p50/p95 latency
- field nRMSE와 `T_max` MAE
- melt boundary Dice와 melt diameter proxy MAE
- worst 5% CTQ error
- process/stack OOD error

validation primary CTQ가 같으면 false escape가 낮은 모델을, 그마저 같으면 latency가 낮은
모델을 선택한다. test split은 선택에 사용하지 않는다.

## 조사 범위

Search 스킬로 neural operator, graph/transformer, physics-informed/점용접의 세 workstream에서
검색 결과 65건을 검토했다. 모델 사실은 NVIDIA 공식 문서·현재 설치 환경을, 점용접 물리는
원 논문과 원 데이터 저장소를 우선했다.
