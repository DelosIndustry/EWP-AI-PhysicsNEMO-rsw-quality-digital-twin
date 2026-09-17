# M3.2 simulation 물리 타당성 gate

## 목적

M3.1은 checksum, group leakage, tensor schema와 에너지 보존을 통과했다. 그러나 이 검사는
solver가 구현한 방정식을 정확히 푸는지 확인할 뿐, 생략한 물리현상까지 정당화하지 않는다.
M3.2는 수치 무결성, sampling 정렬, 물리 모델 적용 범위를 서로 다른 gate로 평가한다.

## 동결 입력

- data version: `RSW-SIM-V2`
- manifest ID: `2e43da1c058aef17a5500145fc92ea241b038457973adbcddb7162c1b39d2b98`
- sample: train 1,024 / validation 256 / test IID 256 / process OOD 256 / corner 128
- M2 physics: constant-property heat conduction, no latent heat, no phase fraction

test 출력은 모델 학습 전에 dataset의 물리 범위를 감사하는 목적으로만 열었다. 이후 test 결과는
architecture, hyperparameter 또는 threshold 선택에 사용하지 않는다.

## 판정 결과

| gate | 결과 | 근거 |
|---|---|---|
| checksum·group leakage | PASS | 모든 checksum 일치, group overlap 0 |
| 수치보존 | PASS | energy balance 최대 `4.1264e-14` |
| 전기–열원 에너지 | PASS | 상대 gap 최대 `5.1890e-16` |
| train–validation sampling | PASS | Tmax KS `0.0771`, energy KS `0.0674` < `0.10` |
| 전체범위 surrogate 학습 | BLOCK | train에 1700 K 초과 7개, 현재 M2에 상변화 없음 |

1700 K 초과 sample/group은 train 7/2, validation 0/0, test IID 0/0, process OOD
12/4, corner OOD 21/6이다. 최고온도는 corner OOD의 `2004.811 K`다.

## 경계값 계약

1700 K는 보수적인 **solver 검토 경계**다. AISI 1010의 확정 solidus, nugget 합격 기준 또는
생산 공정 규격이 아니다. 예시 저탄소강 연구의 solidus `1768 K`, liquidus `1797 K`, 잠열
`270 kJ/kg`을 plausibility reference로만 사용했다. 합금 조성과 물성 출처가 확정되기 전에는
이 값을 품질 label로 변환하지 않는다.

참고: [Numerical Study on the Solidification Microstructure Evolution in Industrial Twin-Roll
Casting of Low-Carbon Steel](https://pmc.ncbi.nlm.nih.gov/articles/PMC12525790/)

## 진단식

임계온도 \(T_0\)의 초과율은

\[
p(T_0)=\frac{1}{N}\sum_{i=1}^{N}\mathbf{1}
\left[T_{\max,i}>T_0\right]
\]

이고, empirical CDF는

\[
F(T)=\frac{1}{N}\sum_{i=1}^{N}\mathbf{1}
\left[T_{\max,i}\le T\right]
\]

이다. train과 validation의 Kolmogorov–Smirnov 거리는

\[
D=\sup_T\left|F_{train}(T)-F_{validation}(T)\right|
\]

로 계산했다. KS는 분포 모양 차이를 보고, Wasserstein 거리는 온도 K 또는 에너지 J 단위의
평균적인 분포 이동을 보여준다.

## 실행과 시각화

```bash
conda activate physicsnemo-study
python scripts/qualify_rsw_space_filling_data.py
```

결과는 `artifacts/m3_2_physical_validity`에 저장한다.

- `temperature_ecdf.png`: split별 Tmax 누적분포와 검토 경계
- `temperature_regime_fractions.png`: 검토·solidus·liquidus 구간별 표본 비율
- `energy_temperature_map.png`: electrical energy와 Tmax의 연관
- `hottest_parameter_profiles.png`: 전체 최고온도 10개 사례의 입력 조합
- `visualization_metadata.json`: 모든 그림의 split, selection, sample ID와 경계 의미

## 다음 구현: M2.1

현재 V2는 삭제하거나 사후 수정하지 않는다. 다음 데이터 버전은 온도 대신 enthalpy를 상태량으로
사용하는 열 solver에서 생성한다.

\[
\frac{\partial H(T)}{\partial t}
=\nabla\cdot\left(k(T)\nabla T\right)+q_{joule},
\qquad
H(T)=\int c_v(T)\,dT+\rho L f_l(T)
\]

여기서 \(f_l\)은 solidus와 liquidus 사이의 액상분율이다. M2.1은 먼저 zero-latent-heat
회귀, 단일-cell enthalpy 해석해, monotonicity, energy balance와 phase-fraction bound를
통과해야 한다. 그 뒤에만 `RSW-SIM-V3`를 생성하고 PhysicsNeMo FNO를 다시 학습한다.
