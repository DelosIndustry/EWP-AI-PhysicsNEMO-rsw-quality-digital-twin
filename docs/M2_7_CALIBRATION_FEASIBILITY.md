# M2.7 가압력 단위·센서 보정 가능성 감사

## 목적

M2.6은 AISI 1010, 약 0.63 mm × 2 stack과 공개 실험의 형상 호환성을 확인했지만,
CSV의 `Force (N)` 값은 약 5–132 범위여서 기존 1.5–6.0 kN 접촉법칙과 겹치지 않았다.
M2.7은 이 차이를 임의 환산으로 없애지 않고 다음 질문을 검증한다.

1. 저장된 force 값의 quantity와 단위가 공개 근거 사이에서 일관적인가?
2. 센서 bench 검사가 전극 tip force까지 추적 가능한 calibration인가?
3. 평균값과 record index만으로 전기 에너지와 시간축을 복원할 수 있는가?
4. 현재 공개 데이터 중 접촉법칙 또는 내부 온도장을 보정할 수 있는 것이 있는가?
5. 없다면 무엇을 최소한 추가 측정해야 하는가?

## 핵심 발견: N과 PSI가 충돌한다

동일 데이터에 대해 공개된 근거가 서로 다르다.

- [Mendeley v3](https://data.mendeley.com/datasets/rwh8kjzdch/3)의 CSV header:
  `Force (N)`
- [Data in Brief 원 데이터 논문](https://doi.org/10.1016/j.dib.2025.111373):
  electrode force를 Newton으로 설명
- [Sensors 후속 논문](https://doi.org/10.3390/s25061744)의 Table 5:
  같은 0–133.53 범위를 `Electrode pressure (PSI)`로 표기

따라서 현재 올바른 상태는 `N`, `PSI` 또는 `kgf` 중 하나를 추측하는 것이 아니라
`conflicted`다. M2.6에서 관측한 force-domain overlap 0은 다음 조건부 문장으로 수정해
해석한다.

> CSV header의 N을 그대로 가정하면 기존 1.5–6.0 kN 범위와 mean/max overlap은 0이다.
> 그러나 단위 근거가 충돌하므로 물리적인 overlap 여부는 아직 결정할 수 없다.

raw 값은 변경하지 않았고 어떤 scale factor도 적용하지 않았다.

## 압력과 힘은 왜 바로 바꿀 수 없는가

압력은 단위 면적당 힘이다.

\[
P = \frac{F}{A}
\]

하지만 공압 실린더 압력에서 전극 tip force를 얻을 때 필요한 면적은 전극 tip 면적이
아니라 실린더 piston의 유효 면적이다. 기구부와 마찰도 포함하면 예시 관계는 다음과 같다.

\[
F_{\mathrm{tip}}
= P_{\mathrm{cylinder}} A_{\mathrm{piston}}
  \lambda_{\mathrm{mechanical}}\eta - F_{\mathrm{friction}}
\]

현재 공개 자료에서는 piston 면적, 기계 전달비, 효율·마찰 보정, load cell 측정 위치가
확인되지 않았다. 그래서 PSI에서 N으로의 변환 gate는 차단된다.

## 알려진 질량 검사의 정확한 의미

Sensors 논문은 266 g 간격의 0–2128 g 질량 9점에 대해 load-cell reading을 보고한다.
M2.7이 재계산한 결과는 다음과 같다.

- 평균 bias: `1.262 g`
- RMSE: `2.828 g`
- 최대 절대 편차: `4.530 g`
- 최대 reference mass의 중력 환산값: `20.869 N`

이는 그 bench mass 범위에서 reading이 질량을 잘 따라갔다는 근거다. 그러나 다음을
제공하지는 않는다.

- archived force 열의 최종 단위
- load cell에서 electrode tip까지의 전달식
- 실린더 압력과 tip force의 관계
- 수 kN 생산 영역까지의 calibration traceability

따라서 `20.869 N`을 CSV 최대값 또는 기존 contact-law force에 맞추는 scale factor로
사용하면 안 된다.

## 10 Hz와 시계열의 한계

논문은 current와 force를 nominal 10 samples/s로 기록했다고 설명한다. 하지만 공개 CSV에는
원 timestamp가 없고 weld 안의 record index만 있다. 균일 10 Hz이며 시작과 끝을 모두
포함한다고 가정하면 duration \(t\)에 대한 예상 표본수는

\[
N_{\mathrm{hyp}} = \operatorname{round}(10t) + 1
\]

이다. 실제 development 396 weld 중 134개는 이 가정과 표본수가 다르다. 그러므로
`record_index / 10`을 실제 시간으로 만들거나 이 값을 solver time grid로 사용하는 것은
금지한다.

또한 weld별 평균 current 하나는 시계열 변동을 잃는다. Jensen 부등식으로

\[
\mathbb{E}[I^2] \geq \mathbb{E}[I]^2
\]

이고, 실제 development record에서
\(\mathbb{E}[I^2]/\mathbb{E}[I]^2\) 중앙값은 `1.029602`다. 이 값은 평균 하나로
축약할 때 current 변동 정보가 사라진다는 진단이다. 전압, 저항, 정확한 timestamp가 없기
때문에 실제 Joule energy 또는 2.96% 에너지 오차라고 해석하지 않는다.

## invalid 값 보존

development 3,378 records에서 비양수 또는 비정상 process 값은 다음과 같다.

- current: 1 record
- force: 14 records

`-99` 같은 sentinel을 평균으로 대체하지 않았다. CSV와 요약표에 invalid count를 남기고,
분포 그림에서만 숫자 histogram의 결측 항목을 제외했다.

## 공개 후보 데이터 감사

Search 스킬로 force/calibration과 대체 데이터 두 영역에서 검색 결과 40건을 검토했다.
웹 catalog 설명과 실제 raw audit는 다른 상태로 관리한다.

| 후보 | 확인 상태 | 허용 용도 | 금지된 주장 |
|---|---|---|---|
| Mendeley v3 | raw 감사, force 단위 충돌, 전압 없음 | force 제외 lab CTQ 기준선 | 접촉·에너지·내부장 보정 |
| [Polito automotive](https://github.com/smartdatapolito/resistance_spot_welding_dataset) | raw 감사, V/I/F가 0–1 정규화 | 별도 불량·OOD 분류 | SI 복원·물리 보정 |
| [IEEE BIW 후보](https://doi.org/10.21227/wfsq-xw87) | catalog상 동기 I/V/F/R, login 필요, raw 미감사 | sample·license 확인 전 없음 | catalog claim의 실측 승격 |
| [Figshare Welding Data](https://doi.org/10.6084/m9.figshare.30529277) | catalog상 115개 dynamic-resistance curve | expulsion benchmark 후보 | I/V/F·에너지 보정 |
| [Al RSW resistance](https://github.com/JanAlexanderZak/al_rsw_resistance) | code 공개, parquet은 저자 요청, 알루미늄 | 방법 참고 | AISI 1010 target 근거 |

검토 후보 중 측정된 내부 온도장 \(T(r,z,t)\)을 제공하는 데이터는 0개다. 따라서 외부
실측은 nugget, pull strength, 불량 label 등의 CTQ 검증에 쓰고, 내부 온도장의 정답은
검증된 solver simulation과 분리한다.

고속 동기 측정의 설계 참고로는 [22MnB5 variable-force 연구](https://doi.org/10.1007/s40194-020-01001-2)의
25.6 kHz V/I/F/displacement 측정과 [50 kHz RSW 연구](https://www.mdpi.com/2075-4701/11/9/1459)를
검토했다. 이 측정률과 공정 force는 설계 참고이며 AISI 1010의 보편 기준이나 recipe로
옮기지 않는다.

## 두 트랙의 판정

### Lab external CTQ track

`guarded_baseline_feasible`이다. 다음 M6.1에서는 force를 기본 feature에서 제외하고
development weld만으로 nugget diameter, pull test, Good/Bad/Explode 기준선을 만든다.
test 99 weld는 모델·threshold 동결 후 한 번만 평가한다.

### Automotive physical-twin track

`blocked_pending_traceable_data_or_coupon_experiment`다. Polito 분류 benchmark는 병렬로 할 수
있지만, 그것을 SI 물리 보정으로 부르지 않는다. IEEE 후보는 사용자가 login으로 sample을
확보한 뒤 license, 단위, timebase, stack metadata를 직접 감사해야 한다.

## 최소 coupon 측정 protocol

이는 학습용 제안이며 산업 recipe 또는 안전 승인 기준이 아니다. 수치 setpoint는 승인된
장비·재료 WPS 범위 안에서만 정해야 한다.

- calibration: current level code 3 × force level code 3 × replicate 3 = 27 weld
- sealed qualification: 독립 schedule 5 × replicate 3 = 15 weld
- 합계: 42 weld for one destructive CTQ stream
- cross-section과 pull test가 모두 파괴시험이면 각각 별도 specimen 계획 필요
- 실제 weld, batch, schedule 단위로 split하고 replicate를 서로 다른 split에 나누지 않음

Tier A는 동기 terminal V/I/F와 독립 CTQ를 측정한다. 이는 effective total contact component
보정에는 도움이 되지만 전극-판재 위·아래와 faying의 세 접촉항을 분리하지 못한다. M2.5의
관측행렬 rank 1, nullity 2 결론은 유지된다.

Tier B에서 계면별 contact parameter를 주장하려면 interface voltage tap, bulk reference
coupon과 별도 물성 측정을 추가해야 한다.

## 산출물과 그림 읽기

- `metrics.json`: 모든 gate와 최종 track 판정
- `calibration_feasibility_manifest.json`: 근거·후보·protocol hash
- `unit_claims.csv`: N/PSI 단위 주장과 source locator
- `development_trace_summary.csv`: weld별 count, invalid, mean/RMS와 Jensen ratio
- `mass_calibration_points.csv`: 논문 질량점 재계산
- `candidate_registry.csv`, `candidate_gates.csv`: 후보와 허용 가능한 주장
- `coupon_plan.csv`: seed 42의 제안 run order
- `measurement_requirements.csv`: 실험 전 필수 metadata
- `AUTHOR_QUESTIONS_DRAFT.md`: 저자 질문 초안이며 자동 전송하지 않음
- `figures/*.png`: trace, 표본수, 평균 정보 손실, 질량 check, 후보 capability, gate 그림 6개

trace 예시는 target-compatible weld를 record count로 정렬한 뒤 균등 위치 네 개를 선택하므로
항상 sample ID `252, 181, 261, 401`이다. x축은 시간이 아니라 record index다.

## 실행

```bash
conda activate physicsnemo-study
python scripts/run_rsw_m2_7_calibration_feasibility.py
python -m pytest tests/test_rsw_calibration_feasibility.py -q
ruff check src/physicsnemo_study/data/rsw_calibration_feasibility.py \
  src/physicsnemo_study/data/rsw_calibration_feasibility_report.py \
  scripts/run_rsw_m2_7_calibration_feasibility.py \
  tests/test_rsw_calibration_feasibility.py
```

정상 결과는 audit check 16개가 모두 PASS하면서도
`automotive_physical_twin_calibration_ready=false`와 `rsw_sim_v3_generation_ready=false`다.
차단을 무시하지 않은 것이 이번 단계의 성공 조건이다.
