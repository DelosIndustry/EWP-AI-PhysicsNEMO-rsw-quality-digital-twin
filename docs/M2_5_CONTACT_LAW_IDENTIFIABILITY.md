# M2.5 상태의존 접촉 법칙과 식별성

## 목적

M2.4는 판재–판재와 전극–판재 접촉저항을 서로 다른 정적 `R'' [Ω·m²]`로 분리했다.
M2.5는 이를 온도와 가압력 상태에 따라 매 시간구간 갱신할 수 있는 명시적 계약으로
확장한다. 동시에 현재 공개 외부 데이터로 무엇을 보정할 수 없는지 수치적으로 증명한다.

## Contact-law 계약

현재 구현한 bounded power-law는 다음과 같다.

\[
R''(T,F)=R''_{ref}
\left(\frac{T}{T_{ref}}\right)^{a_T}
\left(\frac{F}{F_{ref}}\right)^{-a_F}.
\]

각 법칙은 다음 metadata를 반드시 가진다.

- 계면 종류: 판재–판재, 아래 전극–판재, 위 전극–판재
- `stack_id`, `surface_id`
- 기준 온도·가압력·`R''`
- 온도·가압력 exponent
- 온도·가압력 유효범위
- 범위 이탈 정책
- provenance와 calibration status

기본 범위 정책은 `error`다. 범위 밖 입력을 조용히 외삽하지 않는다. `clamp`는 명시적인
민감도 시험에서만 사용할 수 있다. 이번 계수의 status는 `synthetic_sensitivity`이며
실험 보정값이 아니다.

매 시간구간 시작 온도와 force에서 세 계면의 `R''`를 평가한 뒤 M2.4 보존형 전기 solver와
enthalpy 열 solver를 순차 연성한다. 온도·접촉저항·전압·벌크/접촉 발열·전극 흡수 에너지를
모두 history로 남긴다.

## M2.4 회귀와 수치 검증

온도·가압력 exponent를 모두 0으로 만들면 M2.4 정적 접촉저항으로 돌아가야 한다.

- 온도장 최대 절대차: `0 K`
- 전압 최대 절대차: `0 V`
- terminal energy 절대차: `0 J`
- 전체 verification gate: `15/15 PASS`
- nominal 60→120 time-step 최고온도 차이: `2.050 K`
- 최대 전류 보존 상대오차: `3.232e-13`
- 최대 전기 전력수지 상대오차: `8.073e-14`
- 최대 열 에너지수지 상대오차: `4.360e-15`

## 합성 force 민감도

전류는 5.2 kA로 고정하고 force만 바꿨다.

| scenario | force [N] | Tmax [K] | terminal energy [J] | max voltage [V] |
|---|---:|---:|---:|---:|
| low force | 2,000 | 1282.27 | 168.161 | 0.9198 |
| nominal force | 3,500 | 1034.11 | 128.252 | 0.6934 |
| high force | 5,000 | 911.47 | 108.383 | 0.5799 |

양의 force exponent를 둔 합성 법칙에서는 가압력이 커질수록 `R''`가 감소한다. 같은 전류를
유지하는 전압과 입력에너지, 최고온도도 함께 감소했다. 이 표는 구현 방향성 검증이며 실제
점용접기의 정량 force–temperature 관계가 아니다.

## 구조적 식별 불가능성

균일 원통에서 측정하는 전체 동저항은 다음 합만 관측한다.

\[
R_{dynamic}=\frac{V}{I}
=R_{bulk}+R_{faying}+R_{bottom}+R_{top}.
\]

세 접촉항을 자유 parameter로 두면 한 관측의 Jacobian은 `[1, 1, 1]`이다. 관측 수를
늘려도 계면별 독립 정보가 없다면 설계행렬은 다음 상태로 남는다.

- parameter 수: `3`
- rank: `1`
- nullity: `2`

전체 접촉저항은 고정하고 faying 몫을 5–95%로 바꾼 19개 분해를 계산했다.

- terminal voltage 상대 spread: `1.262e-14`
- terminal power 상대 spread: `1.292e-14`
- sheet source power 상대 spread: `0.4622`

즉 단자 `V/I`는 사실상 완전히 같지만 판재에 들어가는 열은 46.2%까지 달라질 수 있다.
전극–판재 접촉에서 발생한 열 일부가 전극으로 흡수되기 때문이다. 따라서 총 동저항을
맞췄다는 사실만으로 내부 열분배가 보정됐다고 주장할 수 없다.

## 외부 데이터 관측성

| quantity | Mendeley | Polito | 허용된 해석 |
|---|---|---|---|
| current | SI 단위 | 정규화 | Mendeley만 절대 전류 사용 가능 |
| voltage | 없음 | 정규화 | 절대 동저항 계산 불가 |
| force | SI 단위 | 정규화 | Mendeley force–CTQ 관계 가능 |
| 재료·두께 | 있음 | 없음 | Mendeley stack grouping 가능 |
| nugget·pull test | 있음 | 없음 | simulation CTQ 외부 평가 가능 |
| fault/category | category | binary fault | 품질 label 평가 가능 |
| 내부 온도장 | 없음 | 없음 | FNO field 정답으로 사용 금지 |
| 계면별 `R''` | 없음 | 없음 | 직접 보정 불가 |

Mendeley에는 전압이 없고 Polito 전압·전류·force는 원래 scaling을 알 수 없는 정규화
신호다. 따라서 Polito의 `V_norm/I_norm`을 물리 저항 또는 상대 저항이라고 해석하는 것도
금지한다.

허용된 외부 데이터 사용은 다음과 같다.

- Mendeley: 전류·force·두께·재료와 nugget/pull/category의 관계 및 외부 CTQ 평가
- Polito: 정규화 시계열 패턴 기반 fault 분류와 OOD 평가

## 재현과 시각화

```bash
conda activate physicsnemo-study
python scripts/run_rsw_m2_5_contact_law.py
python -m pytest tests/test_rsw_contact_law.py \
  tests/test_rsw_contact_law_evaluation.py -q
```

산출물은 `artifacts/m2_5_contact_law_identifiability/`에 생성된다.

- `metrics.json`
- `force_scenario_metrics.csv`
- `identifiability_decompositions.csv`
- `external_observability.csv`
- `nominal_state_dependent_diagnostics.npz`
- `INTERPRETATION.md`
- PNG 6장

## 다음 gate

M2.5로 상태의존 API와 데이터 한계는 명확해졌지만 `RSW-SIM-V3`는 아직 생성하지 않는다.
M2.6에서 공개 근거를 갖춘 목표 강종·두께·도금·전극 조건을 하나의 target stack으로
정의하고, 직접 보정할 수 없는 contact parameter는 범위·prior·민감도로 관리하는 bounded
calibration 설계를 만든다.
