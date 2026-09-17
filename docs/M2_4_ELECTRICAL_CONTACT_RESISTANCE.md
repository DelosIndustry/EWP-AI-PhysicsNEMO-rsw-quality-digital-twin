# M2.4 명시적 전기 접촉저항

## 목적

M2.3까지의 `reference_conductivity_s_m=1.0e6`은 벌크 저항과 접촉저항이 섞인 유효값이었다.
M2.4는 공개 철 자료에서 얻은 300 K 벌크 전도도 `9.4697e6 S/m`를 사용하고,
판재–판재 및 전극–판재 접촉저항을 별도 `R'' [Ω·m²]` 입력으로 분리한다.

이 단계의 접촉값은 실측 보정값이 아니다. 기존 유효 저항을 보존하도록 만든 합성 초기값으로
API, 보존성, 민감도 경로를 검증하는 데만 사용한다.

## 이산화

축 방향 내부 face의 전체 저항은 다음과 같다.

\[
R_f = \frac{1}{A_f}\left(
\frac{\Delta z}{2\sigma_-} + R''_f +
\frac{\Delta z}{2\sigma_+}
\right).
\]

여기서 `R''=0`이면 기존 harmonic-mean finite-volume conductance와 정확히 같아진다.
전극 경계는 인접한 반쪽 cell의 벌크 저항과 전극–판재 `R''`를 직렬로 연결한다.

각 face 전류가 `I_f`일 때 접촉 발열은

\[
P_{c,f}=I_f^2\frac{R''_f}{A_f}
\]

이다. 판재–판재 접촉열은 두 이웃 cell에 절반씩 분배한다. 전극–판재 접촉열은 설정된
비율만 판재에 넣고 나머지는 전극 흡수 에너지로 별도 기록한다.

따라서 매 시간구간에 다음 두 장부가 닫혀야 한다.

\[
P_{terminal}=P_{bulk}+P_{faying}+P_{electrode-contact},
\]

\[
E_{terminal}=E_{sheet-source}+E_{electrode-absorbed}.
\]

## 합성 초기값 구성

기존 300 K 유효 전도도와 두 판재 전체 높이로부터

\[
R''_{old}=L/\sigma_{effective}=2.0\times10^{-9}\ \Omega m^2
\]

를 얻었다. 직접 자료의 벌크 저항 몫은

\[
R''_{bulk}=\rho_{bulk}L=2.112\times10^{-10}\ \Omega m^2
\]

이므로 남은 `1.7888e-9 Ω·m²`를 판재–판재 56%, 아래 전극–판재 22%, 위 전극–판재
22%로 나눴다. 이 비율은 실험 추정치가 아니라 초기화 identity다.

반경 방향으로는 접촉 반경 안쪽의 conductance가 크고 바깥쪽이 작아지는 매끄러운 profile을
사용했다. low/nominal/high는 모든 `R''`에 각각 0.5/1.0/1.5를 곱한 구현 민감도다.

## 검증 결과

- 자동 gate: `12/12 PASS`
- 균일 원통 series 전압 상대오차: `3.936e-15`
- 해석적 접촉전력 상대오차: `8.185e-15`
- `R''=0` 전위 회귀 오차: `1.141e-15`
- `R''=0` 전류 회귀 오차: `9.570e-16`
- 새 보존형 발열–단자전력 gap: `5.981e-16`
- 80→160 time-step nominal 최고온도 차이: `0.474 K`

| scenario | Tmax [K] | terminal energy [J] | contact energy share | max liquid fraction |
|---|---:|---:|---:|---:|
| low | 807.13 | 96.725 | 81.57% | 0 |
| nominal | 1288.80 | 186.849 | 84.20% | 0 |
| high | 1768.18 | 277.011 | 85.06% | 0.00620 |

접촉저항 scale이 커질수록 같은 6.395 kA 전류를 유지하는 데 필요한 전압과 terminal energy,
최고온도가 모두 증가했다. high scenario에서만 고상선을 아주 조금 넘어 상변화 경로도
실행됐다. 6.395 kA 및 세 scale은 보정된 용접 recipe가 아니라 gate 활성화용 조건이다.

기존 M1의 cell-centre gradient 발열 적분은 단자전력과 약 `6.206e-4`의 상대 gap이 있었다.
새 solver는 resistor power를 직접 분배하므로 M2.2에서 사용하던 사후 Joule 정규화가
필요하지 않다.

## 재현

```bash
conda activate physicsnemo-study
python scripts/run_rsw_m2_4_electrical_contact.py
python -m pytest tests/test_rsw_electrical_contact.py \
  tests/test_rsw_contact_electrothermal_coupling.py \
  tests/test_rsw_contact_evaluation.py -q
```

산출물은 `artifacts/m2_4_electrical_contact/`에 저장된다.

- `metrics.json`: gate와 scenario 수치
- `scenario_metrics.csv`: 표 분석용 결과
- `nominal_diagnostics.npz`: 그림을 다시 그릴 수 있는 원시 배열
- `INTERPRETATION.md`: 한국어 해석 순서
- PNG 5장: 접촉 profile, 전력 분해, 시간 이력, 에너지 장부, 공간장

## 남은 차단 조건

- 접촉저항의 온도 의존성
- 전극 가압력 및 접촉반경 의존성
- 표면 거칠기·산화막·도금·강종별 보정
- 공개 실측 전압/전류 시계열과의 identifiability 검토

M2.5에서 `Rpp(T,F,stack,surface)` API와 식별성 감사를 완료했다. 공개 외부 데이터만으로
절대·계면별 Rpp를 보정할 수 없으므로 `RSW-SIM-V3` 차단은 유지한다. M2.6에서 target
steel stack과 contact prior/range를 provenance와 함께 정의한다.
