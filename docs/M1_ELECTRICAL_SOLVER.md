# M1 축대칭 전기 solver

## 목적

점용접 전기-열 연성의 첫 구성요소로 축대칭 전기전도 문제를 푼다.

\[
\frac{1}{r}\frac{\partial}{\partial r}
\left(r\sigma\frac{\partial\phi}{\partial r}\right)
+\frac{\partial}{\partial z}
\left(\sigma\frac{\partial\phi}{\partial z}\right)=0
\]

\[
\mathbf J=-\sigma\nabla\phi,\qquad
q_{\mathrm{joule}}=\mathbf J\cdot\mathbf J/\sigma
\]

`r=0`과 바깥 반경에는 절연 조건을, 위·아래에는 전위 조건을 둔다. 내부 face의
전기전도도는 harmonic mean으로 계산하고 실제 환형 면적을 사용해 전류를 적분한다.

## 실행

```bash
python scripts/run_rsw_m1_electrical.py
```

산출물은 다음 경로에 생성된다.

```text
projects/rsw_quality_digital_twin/artifacts/m1_electrical_solver/
├── metrics.json
├── INTERPRETATION.md
└── figures/
    ├── analytic_validation.png
    ├── electrical_fields.png
    ├── grid_convergence.png
    └── current_scaling.png
```

## 현재 검증 범위

- 균질 원통의 선형 전위 해석해와 총전류를 비교한다.
- 이질 전기전도도에서도 위·아래 전극 전류가 보존되는지 확인한다.
- 모든 cell에서 Joule heat가 음수가 아닌지 확인한다.
- 목표 전류가 2배일 때 Joule power가 4배인지 확인한다.
- 매끄러운 축방향 가변 전기전도도 문제에서 격자 수렴을 확인한다.

이 단계의 형상·물성·계면 저전도 영역은 수치 검증용 합성 조건이다. 실제 자동차
점용접 조건으로 보정된 데이터나 공정 품질 예측 결과로 해석하면 안 된다.
