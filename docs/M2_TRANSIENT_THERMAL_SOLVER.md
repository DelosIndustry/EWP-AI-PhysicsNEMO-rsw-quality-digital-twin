# M2 축대칭 비정상 열 solver

## 방정식과 이산화

M1에서 계산한 Joule heat를 다음 열전도 방정식의 체적 열원으로 사용한다.

\[
\rho c_p\frac{\partial T}{\partial t}
-\frac{1}{r}\frac{\partial}{\partial r}
\left(rk\frac{\partial T}{\partial r}\right)
-\frac{\partial}{\partial z}\left(k\frac{\partial T}{\partial z}\right)
=q_{\mathrm{joule}}
\]

- 공간: cell-centered axisymmetric finite volume
- 시간: 1차 완전 음해법(Backward Euler)
- 재료 face: 열저항 기반 harmonic flux
- 판재 계면: 선택적인 열접촉 전도도 `h_contact [W/(m² K)]`
- 반경 경계: 단열
- 위·아래 전극 경계: 선택적인 대류 냉각

각 시간 간격에서 다음 에너지 수지를 계산한다.

\[
E_{\mathrm{source}}-E_{\mathrm{boundary}}-\Delta E_{\mathrm{stored}}=0
\]

## 실행

```bash
/home/work/sdh/envs/physicsnemo-study/bin/python scripts/run_rsw_m2_thermal.py
```

결과는 다음 위치에 생성된다.

```text
projects/rsw_quality_digital_twin/artifacts/m2_thermal_solver/
├── metrics.json
├── INTERPRETATION.md
└── figures/
    ├── temperature_history.png
    ├── temperature_fields.png
    ├── energy_balance.png
    └── grid_time_convergence.png
```

## 검증 항목

1. 무열원·균일온도 조건에서 온도가 변하지 않는가?
2. 단열·균일발열 조건에서 `T=T0+q*t/(rho*cp)`를 재현하는가?
3. M1 Joule heat를 전달했을 때 이산 에너지 수지가 닫히는가?
4. 통전 종료 뒤 전극 냉각에 의해 최고온도가 감소하는가?
5. 격자와 시간 간격을 줄이면 cosine diffusion 해석해 오차가 감소하는가?

현재 config의 물성과 공정조건은 수치 검증용 합성 값이다. 온도 의존 물성, 잠열,
상변태, 용융 유동과 실제 전극 고체는 포함하지 않으므로 최고온도나 고온영역을 실제
nugget 품질로 해석하면 안 된다.
