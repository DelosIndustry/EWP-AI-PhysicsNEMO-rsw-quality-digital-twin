# M2.3 고온 벌크 전기저항 민감도 gate

## 목적

M2.2는 온도의존 전기–열 되먹임을 구현했지만 옮긴 전기저항 표가 1000 K에서 끝났다.
M2.3은 용융 직전까지 공개된 직접 측정값을 추가하고, 직접 점이 없는 온도 구간을 하나의
임의 곡선으로 확정하지 않고 세 가정으로 나누어 결과 민감도를 측정한다.

이 단계는 벌크 전기저항의 **온도범위와 불확실성 처리**를 검증한다. 목표 자동차 강종이나
접촉저항을 보정하는 단계는 아니다.

## 사용한 직접 자료

### 300–1000 K

NBS Special Publication 260-90 Table 4.1의 electrolytic-iron RRR=20 권장값을 사용했다.
값은 열팽창 보정된 전기저항이며 단위는 `nΩ·m`다.

- 출처: [NBS SP 260-90](https://www.govinfo.gov/content/pkg/GOVPUB-C13-8476c611bb917728f7f6e93844b12e9c/pdf/GOVPUB-C13-8476c611bb917728f7f6e93844b12e9c.pdf)
- 사용 범위: 300–1000 K
- 300 K/1000 K 값: 105.6/909.0 nΩ·m

### 1500–1800 K

Cezairliyan과 McClure가 99.9% 철을 1500–1800 K에서 빠르게 저항 가열하며 측정한 값을
사용했다. 논문은 전기저항 부정확도를 약 1%로 평가하며 1500–1660 K의 γ-iron과
1700–1800 K의 δ-iron을 따로 회귀했다.

- 출처: [NBS Journal of Research 78A](https://nvlpubs.nist.gov/nistpubs/jres/78A/jresv78An1p1_A1b.pdf)
- 사용 범위: 1500–1800 K
- 1500 K/1800 K 값: 1206.6/1268.7 nΩ·m

두 자료 모두 순철 또는 electrolytic iron이다. 탄소, 합금원소, 도금과 실제 판재 제조 이력이
반영된 자동차용 저탄소강 물성으로 간주하지 않는다.

## 1000–1500 K bridge

현재 선택한 두 표 사이에는 직접 사용한 점이 없다. 시작점과 끝점을 보존하면서 다음 식의
지수 `p`만 바꿨다.

\[
s=\frac{T-1000}{1500-1000},
\qquad
\rho(T)=\rho_{1000}+
\left(\rho_{1500}-\rho_{1000}\right)s^p .
\]

| scenario | p | 의미 |
|---|---:|---|
| `early_rise` | 0.5 | gap 초반부터 저항이 빠르게 증가 |
| `linear` | 1.0 | 두 endpoint의 직선 연결 |
| `late_rise` | 2.0 | gap 후반에 저항이 빠르게 증가 |

이 선들은 측정값이 아니라 불확실성 가정이다. 모든 시나리오에서 300–1000 K와
1500–1800 K의 직접 자료는 한 점도 변경하지 않는다.

## 동일한 공정 입력에서의 결과

검증용 전류는 5.7 kA, 통전시간은 40 ms다. 실제 생산 recipe가 아니라 세 곡선이
상변화 구간에서 어떤 차이를 만드는지 보기 위한 synthetic case다.

| scenario | Tmax [K] | 최대 액상분율 | terminal energy [J] | mushy proxy [mm] |
|---|---:|---:|---:|---:|
| early rise | 1774.968 | 0.240279 | 462.994 | 3.0 |
| linear | 1774.541 | 0.225558 | 458.434 | 3.0 |
| late rise | 1774.226 | 0.214696 | 453.803 | 3.0 |

민감도 범위는 다음과 같다.

- 최고온도: `0.742 K`
- terminal energy: `9.191 J`
- 최대 액상분율: `0.025583`
- mushy diameter proxy: `0.0 mm`

온도 차이가 작은 이유는 세 case가 잠열이 온도 상승을 억제하는 mushy 구간에 들어갔기
때문이다. 추가 에너지는 온도보다 액상분율 차이로 더 선명하게 나타난다.

mushy diameter가 모두 3.0 mm인 것은 민감도가 없다는 증거가 아니다. 반경 10-cell 격자에서
직경 proxy가 1 mm 단위로 바뀌기 때문에 세 경우의 작은 경계 이동을 구분하지 못한 것이다.
V3 preflight 전에 radial refinement 검사가 필요하다.

## 수치 검증

9개 gate가 모두 통과했다.

- 직접 측정점 보존
- 세 bridge의 예상 저항 순서와 전체 단조성
- 1800 K curve가 1797 K 액상선까지 덮는지
- 모든 scenario 온도가 curve 범위 안인지
- 전류, terminal–Joule energy와 enthalpy energy 보존
- 높은 저항 scenario가 더 높은 에너지·온도를 내는지
- 상변화 활성화
- 80→160 time-step refinement에서 Tmax 차이 `3.789 K < 5 K`

수치 gate의 PASS는 세 가정 아래 계산이 일관된다는 뜻이며 세 가정 중 하나가 실제 강판의
정답이라는 뜻은 아니다.

## reference conductivity 진단

현재 설정은 M2.2와의 비교를 위해 `1.0e6 S/m`의 유효 reference conductivity를 유지한다.
하지만 300 K 표의 105.6 nΩ·m를 그대로 역수로 바꾸면 약 `9.47e6 S/m`다.

\[
\sigma_{bulk,300}=\frac{1}{105.6\times10^{-9}}
\approx9.47\times10^6\;\mathrm{S/m}.
\]

두 값의 차이는 현재 reference 값이 순수 벌크 물성이 아니라 접촉·유효 저항을 함께 흡수한
scale임을 보여준다. 따라서 V3 전에 벌크저항과 접촉저항을 분리해야 한다.

## 접촉저항 조사 결론

점용접의 접촉저항은 하나의 보편 상수가 아니다. 공개 연구들은 다음 변수에 민감하다고
보고한다.

- 판재–판재와 전극–판재의 서로 다른 계면
- 접촉 압력과 실제 접촉면적
- 온도에 따른 경도·산화막·constriction 변화
- 도금, 표면조도, 오염과 전극 마모

따라서 다른 논문의 계수를 현재 stack에 그대로 대입하지 않는다. M2.4에서
`Rpp [Ω·m²]` 보존형 interface solver와 해석해 검증을 완료했으며, 값은 여전히 합성
초기값이다. 온도·가압력·stack별 계수 법칙과 외부 데이터 식별성은 M2.5로 분리한다.

참고한 1차 연구:

- [Babu et al., pressure-temperature contact resistance model](https://doi.org/10.1179/136217101101538631)
- [Rogeon et al., contact-condition measurements](https://doi.org/10.1016/j.jmatprotec.2007.04.127)
- [Li et al., local contact resistance in RSW](https://doi.org/10.1016/j.ijheatmasstransfer.2012.01.040)

## 실행과 시각화

```bash
conda activate physicsnemo-study
python scripts/run_rsw_m2_3_high_temperature.py
python -m pytest tests/test_rsw_high_temperature_sensitivity.py -q
```

결과는 `artifacts/m2_3_high_temperature_sensitivity/`에 생성된다.

1. `resistivity_evidence_and_bridges.png`: 직접 측정점과 가정 구간 구분
2. `sensitivity_time_histories.png`: 온도·전압이 갈라지는 시점
3. `sensitivity_summary.png`: Tmax·액상분율·에너지·직경 비교
4. `energy_conservation.png`: 전기–열 교차 에너지 보존
5. `peak_field_comparison.png`: 공통 색 범위의 공간장 비교

JSON, CSV와 NPZ에는 각 그림의 원 수치를 함께 저장한다.

## V3 판정

- 벌크 전기저항의 온도범위: 액상선까지 연결 완료
- 1000–1500 K 직접 측정: 미확보
- 목표 자동차 강종 보정: 미완료
- 판재–판재 접촉저항: 미완료
- 전극–판재 접촉저항: 미완료
- 가압력 의존성: 미완료

따라서 상태는 `blocked_grade_and_contact_calibration`이며 아직 RSW-SIM-V3를 생성하지 않는다.
