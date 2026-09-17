# M3 결정론적 simulation dataset

## 목적

M1 전기 solver와 M2 비정상 열 solver를 순차 실행해 PhysicsNeMo 모델이 사용할
시공간 field를 생성한다. 현재 데이터는 solver 학습 경로 검증용 합성 데이터이며 공개
실측 RSW 데이터와 직접 병합하지 않는다.

## 실행

```bash
/home/work/sdh/envs/physicsnemo-study/bin/python scripts/generate_rsw_simulation_data.py
```

이미 생성된 dataset을 의도적으로 다시 만들 때만 `--overwrite`를 사용한다.

## Tensor 계약

```text
inputs/field                       float32 [N, 10, T, R, Z]
targets/temperature_rise_k         float32 [N,  1, T, R, Z]
aux/temperature_k                  float32 [N,  1, T, R, Z]
aux/electric_potential_v           float32 [N,  1, T, R, Z]
aux/joule_heat_w_per_m3            float32 [N,  1, T, R, Z]
```

입력 channel 순서는 manifest의 `tensor_contract.input_channels`가 source of truth다.

## 결정론성

- global seed와 split별 seed offset을 함께 사용한다.
- 각 group과 sample은 독립 RNG stream을 사용한다.
- NPZ member 순서, timestamp와 file permission metadata를 고정한다.
- cell-centered Joule heat는 sample별로 적분값이 `V*I`와 같도록 보존 정규화한다.
- manifest ID는 config, file SHA-256, normalization, summary를 canonical JSON으로 hash한다.
- `created_at_utc`는 manifest ID 계산에서 제외한다.

## Split 누수 방지

각 group은 같은 stack, current schedule family, contact model을 공유하는 4개 sample로
구성한다. `stack_id`, `schedule_id`, `contact_model_id`는 split마다 별도 namespace를
사용하며 생성 직후 모든 pairwise 교집합이 0인지 검사한다.

## 아직 설정하지 않은 값

용융온도는 외부 근거와 M3 온도분포 검토 전까지 `null`이다. 따라서
`melt_diameter_proxy_m`, `time_above_melt_s`는 NPZ에서 `NaN`, manifest에서
`not_configured`로 기록한다. 임의 threshold로 PASS/FAIL label을 만들지 않는다.
