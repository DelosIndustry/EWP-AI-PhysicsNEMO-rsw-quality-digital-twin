# M2.1 enthalpy 기반 상변화 열 solver

## 목적

M2의 상수 물성 열전도는 수치보존을 만족하지만 `RSW-SIM-V2` 일부를 2004.8 K까지 계산했다.
이 온도범위에서 잠열을 생략한 온도장은 nugget 형성용 정량 정답으로 사용할 수 없다. M2.1은
기존 M2를 보존하면서 enthalpy 상태량, 액상분율과 온도의존 열전도도를 별도 solver로 추가한다.

## 지배방정식

cell의 체적 enthalpy를 \(H\,[\mathrm{J/m^3}]\)로 두면

\[
\frac{\partial H(T)}{\partial t}
-\nabla\cdot\left(k(T)\nabla T\right)=q_{joule}
\]

이다. enthalpy와 선형 액상분율은

\[
H(T)=\int_{T_{ref}}^T c_v(\theta)\,d\theta+\rho L f_l(T),
\qquad
f_l(T)=\operatorname{clip}\left(
\frac{T-T_s}{T_l-T_s},0,1
\right)
\]

로 정의한다. \(c_v\)는 체적 sensible heat capacity, \(L\)은 질량당 잠열이다.

Backward Euler finite-volume 잔차는

\[
R(T^{n+1})=
\frac{V}{\Delta t}\left[H(T^{n+1})-H(T^n)\right]
+K(T^{n+1})T^{n+1}-Q-b=0
\]

이고, `dH/dT`를 Jacobian 대각항에 넣은 감쇠 quasi-Newton으로 푼다. 각 반복에서
`k(T)`로 보존형 전도 행렬을 다시 조립한다.

## 물성의 지위

현재 설정의 저탄소강 예시값은 solidus `1768 K`, liquidus `1797 K`, 밀도 `7000 kg/m³`,
비열 `680 J/(kg·K)`, 잠열 `270 kJ/kg`, 고체/액체 열전도도 `28.4/36 W/(m·K)`다.
이는 [공개 저탄소강 연구](https://pmc.ncbi.nlm.nih.gov/articles/PMC12525790/)의
plausibility reference이며 AISI 1010 또는 실제 용접 판재에 보정된 값이 아니다.

## 자동 검증

1. 잠열 0·상수 물성에서 기존 M2 온도장과 일치
2. `H(T)`와 역함수 `T(H)` round trip
3. 단열·균일 발열에서 enthalpy 해석해와 일치
4. 잠열을 포함하면 동일 에너지에서 sensible-only보다 온도가 낮음
5. 액상분율이 `[0, 1]`이고 온도에 따라 단조 증가
6. M1 Joule heat 연성에서 전기–열원 에너지와 열에너지 수지 일치
7. 모든 시간 간격에서 비선형 잔차 수렴
8. 통전 종료 뒤 최고온도 감소

## 실행과 그림

```bash
conda activate physicsnemo-study
python scripts/run_rsw_m2_1_enthalpy.py
```

산출물은 `artifacts/m2_1_enthalpy_solver`에 저장된다.

- `material_enthalpy_properties.png`: enthalpy, apparent heat capacity, `k(T)`
- `uniform_heating_verification.png`: numerical·해석해·잠열 0 반사실 비교
- `coupled_temperature_phase_history.png`: 통전·중심온도·최대 액상분율
- `coupled_temperature_phase_fields.png`: 동일 시각의 온도장·액상분율장
- `energy_nonlinear_convergence.png`: 에너지 수지·반복수·최종 잔차

## 아직 하지 않는 것

- 액체 금속 유동, 전극 가압 변형과 expulsion
- 온도 상승에 따른 전기저항 변화의 양방향 electro-thermal coupling
- 실제 화학조성 기반 solidus/liquidus 계산
- 실측 nugget 직경을 사용한 물성·접촉저항 역보정

M2.1 검증 통과만으로 V3 생성이 자동 승인되지는 않는다. 재료 조성·물성 곡선의 출처와 V3
parameter range를 먼저 동결해야 한다.

## 현재 검증 결과

- 자동 gate: `11/11 PASS`
- 잠열 0 M2 회귀 최대 온도차: `2.842e-13 K`
- 균일가열 enthalpy 해석해 최대 오차: `1.251e-11 K`
- coupled 최고온도/최대 액상분율: `1779.577 K` / `0.399190`
- coupled source energy: `304.068588 J`
- coupled 에너지 수지 최대 상대오차: `1.519e-15`
- 최대 비선형 반복/최종 상대잔차: `5` / `5.307e-10`

이는 수치 구현 gate 통과를 뜻한다. 물성 보정이나 실제 용접 품질 검증 통과를 뜻하지 않는다.
