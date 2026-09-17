# M2.1 enthalpy 상변화 solver 결과 해석

## 결론

- 자동 검증 11/11개가 통과했다.
- 잠열 0 조건에서 기존 M2와 수치적으로 일치한다.
- 상변화 구간의 저장에너지는 apparent-Cp 근사가 아니라 H(T) 차이로 계산한다.
- 물성은 문헌 plausibility reference이며 AISI 1010 또는 공장 공정에 보정되지 않았다.

## coupled case

- 최고온도: 1779.577 K
- 최대 액상분율: 0.399190
- 최종 source energy: 304.068588 J
- 최대 energy-balance error: 1.519e-15
- 최대 nonlinear iteration: 5

## 그림 읽는 법

1. `material_enthalpy_properties.png`: mushy 구간의 잠열과 물성곡선을 본다.
2. `uniform_heating_verification.png`: numerical 점이 enthalpy 해석곡선과 겹쳐야 한다.
3. `coupled_temperature_phase_history.png`: 통전·온도·액상분율의 시간 관계를 본다.
4. `coupled_temperature_phase_fields.png`: 같은 시각의 온도장과 액상분율장을 비교한다.
5. `energy_nonlinear_convergence.png`: 에너지 곡선 일치와 반복수·잔차를 본다.

## 제한

- 액상분율은 solidus-liquidus 사이 선형이며 유동·expulsion을 풀지 않는다.
- 전기장은 열장으로부터 되먹임되지 않는 순차 연성이다.
- 이 결과를 실제 nugget 직경이나 용접 합격 기준으로 사용하지 않는다.
