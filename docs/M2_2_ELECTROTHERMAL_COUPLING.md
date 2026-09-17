# M2.2 온도의존 전기–열 약연성

## 목적

M2.1까지는 한 번 계산한 Joule heat 이력을 열 solver에 넣는 단방향 연성이었다. M2.2는
온도가 전기저항을 바꾸고, 그 변화가 고정 전류 조건의 전압과 발열에 다시 영향을 주는
최소한의 되먹임을 구현한다.

이 단계의 목적은 실제 강종 보정이 아니라 **결합 알고리즘과 보존성의 수치 검증**이다.

## 사용 방정식

전기장은 각 열 시간구간 시작점 `n`의 온도에서 다음을 푼다.

\[
\nabla\cdot\left(\sigma(T^n)\nabla\phi^n\right)=0,
\qquad
\mathbf J^n=-\sigma(T^n)\nabla\phi^n .
\]

전압은 명령 전류가 되도록 선형 scaling한다.

\[
I^n=I_{\mathrm{command}}^n,
\qquad
P_{\mathrm{terminal}}^n=V^n I^n .
\]

cell-center gradient로 계산한 Joule field의 체적 적분은 경계 flux 전력과 아주 작은 이산화
차이를 가질 수 있다. 따라서 공간 패턴은 유지하면서 다음 보존 정규화를 적용한다.

\[
q_{J,*}^n=q_J^n
\frac{V^n I^n}{\sum_c q_{J,c}^n\,\Delta V_c}.
\]

이 열원을 M2.1 enthalpy 식에 한 시간구간 전달한다.

\[
\frac{H(T^{n+1})-H(T^n)}{\Delta t}
=\nabla\cdot\left(k(T^{n+1})\nabla T^{n+1}\right)+q_{J,*}^n.
\]

따라서 현재 방법은 interval-lagged weak coupling이다. 한 구간 안에서 전기장과 열장을
반복하는 strong coupling은 아직 포함하지 않는다.

## 전기 물성의 의미와 제한

설정에는 NIST/NBS Special Publication 260-90 Table 4.1의 electrolytic iron 전기저항
값 300–1000 K를 옮겼다. 사용 방식은 절대 저항률이 아니라 300 K 대비 상대 변화율이다.

\[
\sigma(T)=\frac{\sigma_{\mathrm{ref}}}{\rho(T)/\rho(300\,\mathrm K)}.
\]

- 출처: [NBS Special Publication 260-90](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nbsspecialpublication260-90.pdf)
- 원 보고서의 electrolytic-iron 범위: 약 296–1373 K
- 현재 코드에 옮겨 검증한 표 범위: 300–1000 K
- 사용 지위: `electrolytic_iron_proxy_not_low_carbon_steel_calibration`

탄소 함량, 강종, 접촉 상태의 영향을 보정하지 않았으므로 AISI 1010이나 자동차 판재의 실제
물성이라고 부르면 안 된다. 기본 정책은 `error`이며 300–1000 K 밖에서 자동 외삽하지 않는다.

## 검증 결과

기본 설정에서 수치 검증 15개가 모두 통과했다.

- 상수 전기물성에서 기존 단방향 해와 최대 온도 차이: `1.705e-13 K`
- 300→600 K 균일 도체의 기대 저항비: `3.173295`
- 계산된 고정전류 전압비/전력비 상대오차: 각각 약 `1.12e-15`, `9.80e-16`
- 동적 물성 최고온도: `634.869 K`
- frozen-300 K 물성 최고온도: `575.021 K`
- 동적/frozen 최종 terminal energy: `77.507/51.468 J`
- 최대 전류 보존오차: `4.807e-15`
- 최대 terminal–source energy gap: `1.726e-16`
- 최대 enthalpy energy-balance error: `5.177e-15`

고정 전류에서는 \(P=I^2R\)이므로 온도 상승에 따른 저항 증가는 전력 증가로 이어진다. 이
양의 되먹임을 frozen-property 기준선과의 차이로 확인했다.

## 실행

```bash
conda activate physicsnemo-study
python scripts/run_rsw_m2_2_electrothermal.py
python -m pytest tests/test_rsw_electrothermal_coupling.py -q
```

주요 구현 파일은 다음과 같다.

- `src/physicsnemo_study/physics/rsw_electrothermal_coupling.py`: 물성 범위와 결합 solver
- `src/physicsnemo_study/physics/rsw_electrothermal_evaluation.py`: 검증·그림·NPZ 생성
- `projects/rsw_quality_digital_twin/configs/m2_2_electrothermal.yaml`: 재현 설정

## 결과 해석

`artifacts/m2_2_electrothermal/` 아래에 다음을 저장한다.

- `metrics.json`: 모든 수치와 gate 판정
- `diagnostics.npz`: 시간·전압·전류·온도·전도도·Joule field
- `INTERPRETATION.md`: 한국어 결과 요약
- `figures/electrical_property_coverage.png`: 전기 물성 범위와 상변화 범위의 간극
- `figures/feedback_history.png`: 동적/frozen 온도·전압 비교
- `figures/energy_accounting.png`: terminal, Joule, stored+loss 에너지 비교
- `figures/field_snapshots.png`: 통전 말기의 온도·저항비·발열장

## V3 데이터셋 gate

M2.2 수치 구현은 완료되었지만 `RSW-SIM-V3` 용융 데이터셋은 아직 만들지 않는다. 현재
전기 물성표 상한 1000 K와 열 모델 액상선 1797 K 사이에 797 K의 미검증 범위가 있기 때문이다.

다음 중 하나가 충족되어야 V3 preflight로 넘어간다.

1. 목표 강종의 고온 전기저항을 용융 범위까지 확보하고 provenance를 고정한다.
2. 고온 범위를 명시적 불확실성 구간으로 정의하고 복수 extrapolation 법칙의 민감도 분석을
   사전 등록한다.

접촉저항 역시 현재 공간 multiplier로만 표현되므로, 실제 품질 주장 전에 온도·가압력 의존
접촉저항 모델이 별도 gate로 필요하다.
