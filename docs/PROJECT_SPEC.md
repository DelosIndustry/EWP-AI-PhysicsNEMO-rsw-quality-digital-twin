# 프로젝트 사양서

## 1. 문제 정의

저항 점용접은 판재와 전극에 전류를 흘려 접촉부에 Joule heat를 발생시키는 과도
다중물리 공정이다. 완전한 해석은 전기·열·기계·금속조직과 접촉을 함께 다뤄야 하지만,
첫 MVP는 **준정상 전기전도 + 과도 열전도**만 순차 연성한다.

영역은 회전 대칭이라고 가정하고 좌표를 `(r, z)`로 둔다. 각 시간에서 전위 `phi`는
다음을 만족한다.

```text
(1/r) * d/dr(r * sigma_e * d(phi)/dr)
+ d/dz(sigma_e * d(phi)/dz) = 0
```

```text
J = -sigma_e * grad(phi)
q_joule = J dot J / sigma_e >= 0
```

온도 `T`는 다음 과도 열전도 방정식으로 계산한다.

```text
rho * c_p * d(T)/dt
- (1/r) * d/dr(r * k * d(T)/dr)
- d/dz(k * d(T)/dz)
= q_joule
```

- `sigma_e [S/m]`: 전기전도도
- `J [A/m^2]`: 전류밀도
- `q_joule [W/m^3]`: 체적 Joule heat
- `k [W/(m K)]`: 열전도도
- `rho*c_p [J/(m^3 K)]`: 체적 열용량
- `T [K]`: 절대온도

축 `r=0`에는 대칭조건을 적용한다. 판재 계면은 첫 단계에서 실제 접촉 변형 대신
유효 접촉 전기저항과 열접촉 전도도를 가진 얇은 층으로 표현한다.

## 2. 중요한 모델링 경계

- 전극 가압력은 M0~M4 field 모델에 직접 넣지 않는다. 기계 접촉을 풀지 않는 상태에서
  force를 독립 입력으로 넣으면 물리적 경로가 불명확하기 때문이다.
- 향후 실험 데이터로 `force -> contact radius/contact resistance` 관계를 보정한 뒤에만
  force를 조건으로 추가한다.
- 용융 온도 이상 영역은 실제 nugget이 아니라 `melt diameter proxy`다. 상변태,
  잠열, 용융 유동과 expulsion을 구현하기 전에는 실제 nugget과 동일하다고 주장하지 않는다.
- 첫 물성 범위와 품질 한계는 실제 자동차 공정 사양이 아니다. 출처와 보정 절차 없이
  임의의 산업 기준을 config에 넣지 않는다.

## 3. ML mapping

고정된 시공간 격자에서 공통 논리 mapping은 다음과 같다.

```text
[material fields, interface fields, I(t), T_initial, cooling]
    -> temperature_rise(t, r, z)
```

FNO 입력과 출력은 channels-first tensor다.

```text
x_fno: B x C_in x T x R x Z
y_fno: B x 1    x T x R x Z
```

Transolver는 같은 sample을 point token으로 바꾼다.

```text
x_token: B x (T*R*Z) x C_in
coords:  B x (T*R*Z) x 3       # normalized t, r, z
y_token: B x (T*R*Z) x 1
```

두 모델은 같은 split, train-only 정규화, loss와 CTQ 평가기를 사용한다.

## 4. 품질 특성

시뮬레이션 온도장에서 다음 값을 계산한다.

- `t_max_k`: 전체 시공간 최고 온도
- `melt_diameter_proxy_m`: 판재 계면에서 용융 온도 이상인 최대 직경
- `time_above_melt_s`: 계면 관심영역이 용융 온도 이상인 누적 시간
- `cooling_rate_k_per_s`: 통전 종료 후 관심지점 냉각속도
- `electrical_energy_j`: `integral(V(t) * I(t) dt)`
- `relative_energy_balance_error`: 입력 전기 에너지와 저장·방출 열의 상대 오차
- `physics_residual`: 전기전도와 열전도의 이산 residual

실험 데이터의 `nugget_diameter`, `pull_force`, `good/bad/expulsion`은 위 시뮬레이션 CTQ와
같은 값으로 간주하지 않는다. 별도 adapter와 보정 실험에서 관계를 평가한다.

## 5. 품질 gate

초기 판정 구조는 기존 열 품질 프로젝트의 원칙을 재사용한다.

```text
if input_is_ood or numerical_check_failed:
    REVIEW
elif confidence_interval_is_safely_inside_learning_limits:
    PASS
elif confidence_interval_is_safely_outside_learning_limits:
    FAIL
else:
    REVIEW
```

`PASS/FAIL` 한계값은 M2 solver와 M6 외부 데이터 검토 전에는 설정하지 않는다. 학습용
한계값을 설정한 뒤에도 실제 양산 판정 기준이라고 표현하지 않는다.

## 6. 모델 선택 원칙

- FNO를 기본 결론으로 정하지 않는다.
- M4에서 CNN, FNO3D, Transolver를 동일 예산으로 비교한다.
- 주 선택 metric은 validation `melt_diameter_proxy` MAE다.
- field nRMSE, `T_max` MAE, 최악조건 오차, latency와 GPU memory를 함께 보고한다.
- MeshGraphNet은 비정형 FEM mesh가 준비된 뒤 별도 실험으로 활성화한다.
- PINO/PhysicsInformer는 모델이 아니라 physics residual을 추가하는 학습 전략으로 다룬다.

## 7. 성공 조건

### 소프트웨어

- FNO3D와 Transolver가 명시된 tensor 계약으로 forward/backward를 통과한다.
- 전기·열 solver의 단위 test와 수렴 test가 ML 학습 전에 통과한다.
- 같은 seed와 config에서 같은 데이터 hash와 `manifest_id`가 생성된다.
- simulation과 experimental provenance가 서로 다른 manifest로 저장된다.
- validation에서 정한 모델·threshold를 test에서 바꾸지 않는다.

### 모델

- 64개 simulation sample overfit에서 field nRMSE `<= 2%`를 우선 확인한다.
- 기준 모델은 CNN 대비 primary CTQ MAE를 `>= 20%` 줄이는 것을 초기 상대 목표로 한다.
- 절대 오차 목표는 M2 solver의 온도·CTQ 분포를 본 뒤 근거와 함께 설정한다.
- 외부 실험 성능이 낮아도 숨기지 않고 sim-to-real gap의 원인을 보고한다.

## 8. 확장 순서

1. 온도 의존 물성 및 유효 잠열
2. `force -> contact state` 보정
3. 전극 고체와 냉각을 포함한 3D geometry
4. 비정형 mesh의 MeshGraphNet/Transolver 비교
5. 열-기계 접촉 및 전극 변형
6. PLC·센서 시계열 모델과 field surrogate의 이중 경로
