# M6.1 external CTQ baseline 해석

## 결론

- 구현·데이터 계약 gate: `20/20 PASS`
- 내부 train/validation weld: `315` / `78`
- 외부 test outcome 사용: `false`
- force feature 사용: `false`
- physical-twin 및 RSW-SIM-V3 준비: `false / false`

## 선택된 validation 모델

- nugget diameter: `random_forest`, MAE `0.2324 mm`, R² `0.3752`
- pull test: `random_forest`, MAE `174.15 N`, R² `0.6829`
- category: `random_forest_balanced`, macro-F1 `0.6813`, balanced accuracy `0.6175`, accuracy `0.9231`
- mean 대비 MAE 감소: nugget `21.22%`, pull test `42.30%`
- prior 대비 category macro-F1 증가: `0.3660`
- category recall (Bad / Explode / Good): `0.667 / 0.200 / 0.986`

accuracy는 Good이 많은 불균형 데이터에서 과대평가될 수 있다. category 선택은 macro-F1을 우선하고, confusion matrix와 Bad/Explode recall을 같이 본다.
특히 0.60–0.66 mm validation subgroup은 Bad 1개, Explode 4개, Good 65개뿐이다. subgroup score는 방향 확인용이며 안정적인 일반화 추정치가 아니다.

## 해석 제한

이 모델은 공개 lab CTQ의 통계 기준선이며 PhysicsNeMo 온도장 모델이 아니다. pressure는 공압 설정값으로만 사용했고 electrode tip force로 해석하지 않았다. record index로 실제 timestamp나 Joule energy도 만들지 않았다.

validation은 개발 데이터 안의 interpolation 성능이다. 후보 세 개 중 하나를 고르는 데 이미 사용했으므로 최종 일반화 성능이 아니다. selection manifest와 model hash를 고정한 다음 단계에서만 외부 test를 한 번 평가한다.
