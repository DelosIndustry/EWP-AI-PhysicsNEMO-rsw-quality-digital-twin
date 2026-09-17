# 점용접 데이터 명세

## 1. 데이터 계층

| 계층 | 출처 | 역할 | 직접 병합 여부 |
|---|---|---|---|
| `simulation_field` | 저장소의 전기-열 solver | PhysicsNeMo field 학습·IID/OOD 평가 | 주 학습 데이터 |
| `experimental_ctq` | Mendeley 공개 실험 | nugget·pull force·열화상 외부 평가 | field target과 병합 금지 |
| `factory_timeseries` | Politecnico di Torino 공개 자료 | 자동차 공정 불량·OOD 별도 평가 | field target과 병합 금지 |

각 계층은 다른 `manifest_id`, license, split과 metric 표를 가진다. 세 결과를 하나의 평균
성능으로 합치지 않는다.

## 2. 시뮬레이션 tensor

공통 원본 tensor는 다음과 같다.

```text
inputs/field                         float32 [N, 10, Nt, Nr, Nz]
targets/temperature_rise_k           float32 [N,  1, Nt, Nr, Nz]
aux/electric_potential_v             float32 [N,  1, Nt, Nr, Nz]
aux/joule_heat_w_per_m3              float32 [N,  1, Nt, Nr, Nz]
metadata/stack_id                    int64   [N]
metadata/schedule_id                 int64   [N]
metadata/contact_model_id            int64   [N]
metadata/distribution                string  [N]
metadata/is_ood                      bool    [N]
metadata/seed                        int64   [N]
ctq/t_max_k                          float32 [N]
ctq/melt_diameter_proxy_m            float32 [N]
ctq/time_above_melt_s                float32 [N]
ctq/cooling_rate_k_per_s             float32 [N]
ctq/electrical_energy_j              float32 [N]
ctq/relative_energy_balance_error    float32 [N]
```

`inputs/field` channel 순서는 config가 source of truth다.

| channel | 시간 의존성 | 공간 표현 |
|---|---|---|
| 전기전도도 | MVP에서는 정적 | 재료 영역별 field |
| 열전도도 | MVP에서는 정적 | 재료 영역별 field |
| 체적 열용량 | MVP에서는 정적 | 재료 영역별 field |
| 판재 mask | 정적 | binary field |
| 계면 mask | 정적 | binary field |
| 전류 schedule | 동적 | 각 시간값을 공간에 broadcast |
| 접촉 비저항 | MVP에서는 정적 | 계면 유효층 field |
| 열접촉 전도도 | MVP에서는 정적 | 계면 유효층 field |
| 초기온도 | 정적 | 전체 영역에 broadcast |
| 냉각계수 | 정적 | 경계 mask와 결합한 field |

모델 입력은 정규화 tensor이고 CTQ와 residual은 반드시 SI 단위로 복원한 tensor에서
계산한다.

## 3. split

개별 time step이나 mesh cell을 무작위로 나누지 않는다.

- `stack_id`: 재료 조합과 판재 두께 topology가 같은 base stack
- `schedule_id`: 동일한 전류 pulse 형태와 통전시간 계열
- `contact_model_id`: 접촉저항·접촉반경 가정이 같은 계열

하나의 ID group은 정확히 하나의 split에만 속한다. OOD 예시는 다음과 같다.

- `process_ood`: train 밖의 pulse 형태, contact resistance 또는 냉각조건
- `stack_ood`: 보지 못한 두께·재료 조합
- `corner`: 높은 전류와 높은 저항 등 극단 조합

smoke는 train 64개로 시작하고 solver·shape·overfit이 통과한 뒤 1,024개 이상으로
확대한다. test 결과를 본 뒤 범위를 바꾸면 새 데이터 버전을 만든다.

## 4. 외부 데이터 split

### Mendeley 실험

- simulation 정규화 통계를 사용하지 않는다.
- `good/bad/expulsion` 비율을 보존해 development/test를 분리한다.
- 접촉저항처럼 관측되지 않은 값을 test label을 보며 맞추지 않는다.
- 열화상은 용접 직후 표면 관측이며 내부 온도장의 정답으로 쓰지 않는다.

### 자동차 공장 시계열

- 같은 `Car Body`가 train과 test에 동시에 들어가지 않도록 group split한다.
- 날짜를 이용한 시간 순서 test를 별도로 둔다.
- 공개 자료의 0~1 정규화 신호는 물리 단위 전류·전압으로 역변환하지 않는다.
- 불량 79개와 정상 1,897개의 불균형을 accuracy로 숨기지 않고 PR-AUC와 fault recall을
  보고한다.

## 5. 저장 형식과 manifest

- smoke 데이터는 deterministic compressed NPZ를 허용한다.
- 전체 시공간 데이터는 HDF5와 Zarr의 읽기 throughput을 비교한 뒤 하나를 고른다.
- 큰 원본 데이터, 외부 archive와 checkpoint는 Git에 commit하지 않는다.

각 manifest는 다음을 기록한다.

- schema/generator/solver version과 Git commit
- seed, 물리 단위, 격자와 time step
- parameter sampling 방법과 범위 출처
- material/contact model 출처
- split별 group ID와 sample 수
- 파일 hash
- 외부 데이터 URL, DOI/version, license와 원본 checksum
- 변환 코드 version과 변환 후 hash

## 6. 누수·안전 규칙

- train 통계만으로 모든 simulation split을 정규화한다.
- 외부 test를 보고 contact model이나 품질 threshold를 바꾸지 않는다.
- license가 불명확한 원본은 저장소에 재배포하지 않는다.
- pickle은 신뢰되지 않은 입력으로 간주하며 가능하면 CSV/NPZ/HDF5로 변환한다.
- 실제 공장명·설비명·차종을 추정하거나 복원하지 않는다.
