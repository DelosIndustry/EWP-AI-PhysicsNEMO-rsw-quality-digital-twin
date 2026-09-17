# M7.1 결과 해석 — 개발 데이터 전용 품질 게이트

## 한 줄 결론

이 결과는 **자동차 양산 승인 기준이 아니라**, 측정된 개발 용접 393개에서
`PASS / REVIEW / FAIL` 구조가 어떤 trade-off를 만드는지 확인한 학습용 feasibility 결과다.
외부 test 99개의 결과는 임계값·모델·OOD 규칙 선택에 사용하지 않았다.

## 가장 먼저 볼 숫자

- 자동 판정 비율(auto coverage): 0.087
- 사람이 확인해야 하는 비율(review rate): 0.913
- 실제 Bad/Explode를 PASS로 보낸 비율(false escape): 0.024
- 실제 Good을 FAIL로 버린 비율(false reject): 0.000
- Bad 안전 포착률(PASS가 아닌 비율): 1.000
- Explode 안전 포착률(PASS가 아닌 비율): 0.960

`false escape`는 불량을 정상으로 통과시키는 오류라 안전 관점에서 가장 중요하다. 반대로
`false reject`는 정상품을 불량으로 버리는 오류라 생산비와 가동률에 영향을 준다. REVIEW는
두 오류를 줄이는 대신 자동화율을 낮추는 완충 지대다.

## 분류와 확률 보정

보정 후 교차예측 3-class balanced accuracy는 0.606, macro F1은
0.621이다. 다수 클래스 Good만 잘 맞히는지 확인하려면 accuracy보다
클래스별 recall과 confusion matrix를 먼저 봐야 한다.

실패 확률 Brier score는 raw RF 0.0734에서
교차 보정 후 0.0604이다. Brier score는 0에
가까울수록 좋다. 단, 희소한 Bad/Explode와 작은 표본 때문에 reliability plot의 각 bin 표본 수도
같이 확인해야 한다.

## OOD 해석 제한

입력 범위 flag 비율은 0.025, Mahalanobis flag 비율은
0.031, 합집합은 0.046이다. 이것은
"학습 분포에서 멀어 보인다"는 경고일 뿐 실제 OOD 정답이 아니다. 알려진 OOD label이 없으므로
OOD recall은 계산하지 않았고, 두 검출기 중 하나를 우승자로 선택하지도 않았다.

## 그림 읽는 순서

1. `01_probability_distributions.png`: Bad/Explode의 P(Good)가 낮게 분리되는지 본다.
2. `02_failure_reliability.png`: 예측 실패확률과 실제 실패빈도가 대각선에 가까운지 본다.
3. `03_decisions_by_category.png`: 각 실제 종류가 PASS/REVIEW/FAIL로 어디에 갔는지 본다.
4. `04_policy_tradeoff.png`: REVIEW를 늘려 false escape를 얼마나 낮추는지 비교한다.
5. `05_ood_diagnostics.png`: 입력 범위와 다변량 거리 경고가 어디에 집중되는지 본다.
6. `06_category_confusion.png`: Bad, Explode, Good의 오분류 방향을 본다.
7. `07_fold_thresholds.png`: fold마다 임계값이 심하게 흔들리는지 본다.

## 아직 부족한 부분

1. 비용 20:5:1은 설명용 가정이며 현대자동차의 실제 비용·품질 요구가 아니다.
2. 임계값은 개발 데이터 안쪽 fold에서 관측 오류 0을 맞춘 값이지, 미래 오류 0의 보장이 아니다.
3. OOD 정답이 없어 OOD recall이나 최적 detector를 주장할 수 없다.
4. Polito 공장 시계열은 Mendeley category와 의미·단위가 달라 M7.2에서 별도 Fault 문제로 다룬다.
5. 이 gate는 측정 CTQ surrogate이며 PhysicsNeMo 온도장 모델의 실측 검증을 대신하지 않는다.
6. 실제 양산 적용 전에는 차종·강종·두께·전극 마모·설비별 시간 순서 외부 검증과
   사전 품질 비용이 필요하다.
