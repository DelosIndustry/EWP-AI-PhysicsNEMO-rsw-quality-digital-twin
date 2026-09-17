# M2.6 목표 steel stack·bounded calibration preflight

## 목적

M2.5까지는 전기·열·상변화·접촉법칙의 수치 계약을 검증했지만 특정 판재와 공정에
보정하지 않았다. M2.6은 공개 실험 데이터와 직접 비교 가능한 target stack 하나를 정하고,
각 입력을 `fixed`, `bounded`, `nuisance`로 나누며 RSW-SIM-V3 생성 가능 여부를 자동
판정한다.

## 목표 stack

`AISI1010_0p63x2_surfaceUnknown_CuAlloyTip3p175`

- 판재: AISI 1010 carbon steel, 2장
- 대표 두께: 각 0.63 mm, compatibility window 0.60–0.66 mm
- 전극: 공개 논문이 보고한 generic copper alloy
- tip contact diameter: 1/8 inch = 3.175 mm
- 도금/표면 처리: 공개 논문과 CSV에 없어 `unknown`
- 실제 electrical contact radius: tip 반경과 같다고 단정할 수 없어 `nuisance`

강종, per-weld 두께, 전극 재료의 일반명과 tip 직경은 공개된
[Data in Brief 원 논문](https://pmc.ncbi.nlm.nih.gov/articles/PMC11893339/)에서 가져왔다.
이 실험은 소형 spot welder에서 수행됐으므로 자동차 양산 stack이라고 부르지 않는다.

## 물성 provenance

- 0.1% carbon-steel의 273.15–1073.15 K 열전도도 표는
  [Wyczółkowski et al.](https://doi.org/10.24425/ather.2022.143170)을 사용했다.
  논문은 표의 측정 불확실도를 약 5%로 두지만, 1073.15 K 위는 직접 근거가 아니므로
  screening 범위는 ±15%로 넓혔다.
- 벌크 전기저항 온도의존성은 기존 M2.3의 NBS electrolytic-iron 표를 유지한다. 이는
  AISI 1010의 절대 물성이 아니라 `proxy`다.
- 비열·밀도·solidus/liquidus·잠열은 정확한 coupon 화학조성이 없으므로 기존 저탄소강
  proxy와 [composition 기반 steel property framework](https://doi.org/10.2355/isijinternational.ISIJINT-2015-365)를 연결했다.
- 계면저항 절대값과 force/temperature exponent는 M2.4/M2.5의 sensitivity prior다.
  target coupon 측정값으로 승격하지 않는다.

모든 상태와 범위는 생성되는 `target_stack_manifest.json`, `evidence_inventory.csv`,
`parameter_registry.csv`에서 확인할 수 있다.

## 외부 development audit

Mendeley test outcome은 선택에 사용하지 않고 development 396 weld만 집계했다.

| 단계 | weld 수 |
|---|---:|
| development | 396 |
| AISI 1010 material match | 396 |
| 두 판 모두 0.60–0.66 mm | 361 |
| 정적 충돌이 없는 stack-compatible | 359 |
| current/force/time 양수 | 358 |
| M2.5 contact-law force 1.5–6.0 kN와 겹침 | 0 |

호환 358개 weld의 force 중앙값은 약 94.99 N이고 최대도 약 116.05 N다. 따라서 기존
contact law를 이 데이터에 외삽해 보정하면 안 된다. 판재 형상 호환성과 공정/법칙 호환성은
서로 다른 문제다.

## OAT screening 결과

실험 recipe가 아닌 3.8 kA, 3.5 kN, 40 ms의 짧은 pulse를 사용해 M2.5 유효범위 안에서
한 번에 한 변수만 low/high로 바꿨다. 총 case는 baseline 1개 + 9변수×2 = 19개다.

- baseline Tmax: 1327.954 K
- baseline terminal energy: 110.919 J
- 전체 case 최고온도: 1774.719 K, 1800 K fail-closed 범위 안
- 최대 보존 상대오차: 8.283e-13

Tmax low–high span이 큰 순서는 다음과 같다.

| parameter | role | Tmax span [K] |
|---|---|---:|
| faying resistance scale | nuisance | 899.705 |
| screening peak current | nuisance | 466.500 |
| effective contact radius | nuisance | 442.462 |
| electrode resistance scale | nuisance | 240.832 |
| volumetric heat capacity scale | bounded proxy | 141.996 |
| thermal conductivity scale | bounded family evidence | 105.104 |
| sheet thickness | bounded direct target | 69.087 |
| bulk resistivity scale | bounded proxy | 22.421 |
| latent heat scale | bounded proxy | 0.000 |

계면저항이 1위라는 결과는 계면저항의 실제 중요도를 정량 확정한 것이 아니다. 설정한 prior가
넓고, 현재 관측으로 좁힐 수 없기 때문에 출력 불확실성을 지배한다는 뜻이다. 잠열 span이 0인
이유는 대부분의 case가 solidus 1768 K 아래였기 때문이다. 이 screening만으로 잠열 prior를
줄일 수 없다.

## 판정

수치·데이터 계약 11개는 모두 통과했지만 `v3_generation_ready=false`다.

차단 원인은 다음과 같다.

1. surface coating/condition과 effective electrical contact radius가 unknown이다.
2. bulk electrical property와 phase property가 target-grade 직접 측정이 아닌 proxy다.
3. faying/electrode contact resistance와 exponent가 식별 불가능한 nuisance다.
4. Mendeley force와 기존 contact-law validity가 겹치지 않는다.

즉, `all_checks_passed=true`는 차단 조건까지 올바르게 감지했다는 뜻이지 보정이나 V3 release가
완료됐다는 뜻이 아니다.

## 실행과 그림 읽기

```bash
conda activate physicsnemo-study
python scripts/run_rsw_m2_6_target_stack.py
python -m pytest tests/test_rsw_target_stack.py -q
```

- `figures/evidence_status.png`: direct/family/proxy/assumption/unknown 분리
- `figures/development_overlap.png`: 396→359 funnel과 force-domain 불일치
- `figures/parameter_registry.png`: bounded와 nuisance 범위
- `figures/screening_tmax_sensitivity.png`: Tmax OAT tornado plot
- `figures/screening_response_matrix.png`: Tmax·energy·liquid fraction span
- `figures/v3_gate_matrix.png`: 수치 PASS와 물리 release BLOCK의 분리

CSV가 모든 그림의 원자료이고 NPZ에는 case별 온도·액상분율·terminal-energy 시간이력이 있다.
