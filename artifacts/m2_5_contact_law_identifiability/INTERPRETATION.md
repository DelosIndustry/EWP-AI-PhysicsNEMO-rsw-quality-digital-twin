# M2.5 접촉 법칙·식별성 결과 해석

## 결론

- 자동 검증 15/15개가 통과했다.
- 온도·가압력 상태를 매 시간구간의 Rpp에 반영하는 API가 동작한다.
- 현재 power-law 계수는 합성 민감도이며 실측 calibration이 아니다.
- 관측행렬 rank/nullity: 1/2.
- Mendeley와 Polito 어느 쪽도 절대·계면분리 Rpp를 직접 식별하지 못한다.
- V3 준비 상태는 `blocked_target_stack_and_contact_calibration`다.

## force sensitivity

- low force Tmax: 1282.27 K
- nominal force Tmax: 1034.11 K
- high force Tmax: 911.47 K

설정한 합성 법칙에서는 force가 커질수록 Rpp가 감소한다. 같은 전류에서는 필요한
전압·입력에너지·최고온도도 낮아진다. 이는 API 방향성 시험이지 실제 stack의 정량
예측이 아니다.

## identifiability 그림

`identifiability_decomposition.png`의 왼쪽 전압은 거의 수평이다. 전체 contact Rpp의
합이 같으면 faying/electrode 배분이 달라도 V/I가 같기 때문이다. 오른쪽 sheet source는
달라진다. 전극 접촉열의 일부가 전극으로 빠진다는 모델 때문에 같은 V/I에서도 판재
온도 결과가 달라질 수 있다.

## 외부 데이터 허용 용도

- Mendeley: SI 전류·가압력·두께와 nugget/pull/category의 CTQ 관계 평가
- Polito: 정규화 시계열 패턴을 이용한 fault 분류와 OOD 평가
- 금지: 두 데이터에서 절대 ohm 또는 Rpp를 복원했다는 주장
- 금지: 내부 온도장을 외부 실측 정답으로 취급

## 그림 읽는 순서

1. `contact_law_curves.png`: 합성 T/F 법칙의 방향과 유효범위를 본다.
2. `force_sensitivity_histories.png`: 같은 전류에서 force 영향만 비교한다.
3. `evaluated_contact_histories.png`: solver가 실제 사용한 Rpp 이력을 본다.
4. `identifiability_decomposition.png`: 같은 V/I가 다른 열분배를 숨김을 본다.
5. `external_observability.png`: 두 외부 데이터의 직접/간접/부재 관측을 본다.
6. `energy_conservation.png`: 상태 법칙을 넣어도 에너지 수지가 닫힘을 확인한다.
