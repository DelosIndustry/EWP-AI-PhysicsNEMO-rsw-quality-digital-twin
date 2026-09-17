# M6.2 frozen external test 해석

## 결론

- 구현·무결성 gate: `27/27 PASS`
- 동결 selection manifest: `174006ad63c751f58f40a38b8a4c27dbf7f110727f4f0c854b0f29801be6fd01`
- 재학습·모델 재선택·threshold 조정: `모두 수행하지 않음`
- 외부 test feature-eligible weld: `98`
- 자동 품질 합격 및 physical-twin 주장: `차단 유지`

## 외부 test 결과

- nugget diameter: MAE `0.2368 mm`, RMSE `0.3367 mm`, R² `0.2055`
- pull test: MAE `147.62 N`, RMSE `243.57 N`, R² `0.5830`
- category: accuracy `0.8878`, macro-F1 `0.6003`, balanced accuracy `0.5682`
- class recall Bad / Explode / Good: `0.750 / 0.000 / 0.955`
- 실제 test class 수 Bad / Explode / Good: `4 / 6 / 88`

## 95% bootstrap 불확실성

- nugget MAE: `0.2368` [`0.1944`, `0.2906`] mm
- pull-test MAE: `147.62` [`112.52`, `187.98`] N
- category macro-F1: `0.6003` [`0.4479`, `0.6538`]

회귀는 weld를 복원추출하는 ordinary bootstrap을 사용했다. 분류는 각 class의 매우 작은 표본수가 bootstrap 반복마다 사라지지 않도록 class별 weld 수를 고정한 stratified bootstrap을 사용했다. 이 방법도 희소 class의 정보 자체를 늘리지는 않는다.

## 현재 부족한 부분

1. 외부 test가 99 weld뿐이며 Bad `4`개, Explode `6`개라 불량 recall의 통계적 불확실성이 크다.
2. 목표 0.60–0.66 mm stack 표본은 `93`개뿐이다. 이 subgroup 결과를 자동차 차체 공정 전체로 일반화할 수 없다.
3. 공개 데이터의 pressure는 공압 설정값이지 electrode tip force 실측값이 아니다. force를 배제했기 때문에 실제 가압력 변화에 대한 인과 설명은 할 수 없다.
4. 공개 데이터에는 내부 온도장 T(r,z,t)가 없다. 이번 결과는 Random Forest CTQ 기준선의 외부 평가이며 PhysicsNeMo 온도장 정확도 검증이 아니다.
5. 한 연구실 데이터셋의 무작위 weld holdout이다. 다른 장비·재료·공장·시간대에 대한 cross-site 또는 temporal validation이 없다.
6. Random Forest의 입력 범위 밖 extrapolation과 OOD를 정식으로 판정하지 않는다. M7에서 REVIEW 경로와 OOD detector를 별도로 검증해야 한다.
7. false escape 비용, 품질 threshold와 확률 calibration이 아직 정의되지 않았다. 따라서 높은 accuracy만으로 자동 합격을 허용하지 않는다.

## 해석 규칙

이 test는 동결 모델의 일반화 성능을 보고하기 위해 사용되었다. 결과를 보고 같은 test에 맞춰 모델, feature, hyperparameter 또는 threshold를 바꾸면 안 된다. 개선이 필요하면 새 development 실험을 만들고, 최종 확인용으로 독립된 새 test 데이터를 확보해야 한다.
