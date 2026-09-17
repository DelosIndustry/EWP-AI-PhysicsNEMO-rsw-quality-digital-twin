# M3.2 simulation 물리 타당성 해석

## 결론

- 원본 manifest `2e43da1c058aef17a5500145fc92ea241b038457973adbcddb7162c1b39d2b98`의 checksum과 수치보존 gate는 통과했다.
- train-validation 분포 gate는 `pass`다.
- 1700 K 초과는 train 7개, process OOD 12개, corner OOD 21개다.
- 현재 solver 전체범위 학습 gate는 `blocked_phase_change_model_required`다.
- 원인은 수치 발산이 아니라 상변화·잠열·온도의존 물성이 없는 모델 범위다.

## 경계값의 의미

- 1700 K는 보수적인 **모델 검토 경계**이며 생산 품질 규격이나 AISI 1010의 보정된 solidus가 아니다.
- 문헌의 예시 저탄소강 값 1768/1797 K는 plausibility reference로만 사용했다.
- 출처: [Numerical Study on the Solidification Microstructure Evolution in Industrial Twin-Roll Casting of Low-Carbon Steel](https://pmc.ncbi.nlm.nih.gov/articles/PMC12525790/)

## 사용한 진단식

- 초과율: $p(T_0)=N^{-1}\sum_i \mathbf{1}[T_{max,i}>T_0]$
- ECDF: $F(T)=N^{-1}\sum_i \mathbf{1}[T_{max,i}\le T]$
- train-validation KS: $D=\sup_T |F_{train}(T)-F_{validation}(T)|$
- KS는 분포 모양 차이, Wasserstein은 물리 단위(K 또는 J)의 평균 이동 크기를 본다.

## 그림 읽는 법

1. `temperature_ecdf.png`: 각 split의 Tmax 누적분포와 검토 경계를 본다.
2. `temperature_regime_fractions.png`: 상변화 모델이 필요한 표본 비율을 본다.
3. `energy_temperature_map.png`: 에너지 증가와 고온 노출의 연관을 본다.
4. `hottest_parameter_profiles.png`: 최고온도 사례의 입력 조합을 본다.

## 다음 결정

- RSW-SIM-V2 원본은 provenance를 위해 그대로 보존한다.
- 이 V2 전체를 nugget 품질용 정량 surrogate의 정답으로 사용하지 않는다.
- 다음은 M2.1 enthalpy 기반 상변화와 온도의존 물성을 구현한 뒤 V3를 생성하는 것이다.
- 현재 그림의 상관관계를 인과관계 또는 실제 공장 품질 규격으로 해석하지 않는다.
