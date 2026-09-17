# M5.1 reference residual 해석

## 판정

**PASS** — train·validation M3 정답장에 M2와 동일한 이산 residual을
적용한 최악의 등가 온도 오차는 `3.510684e-04 K`이다. 사전 고정 기준은
`1.0e-03 K`이다.

## 의미

- 정답장의 초기 온도 상승은 정확히 0 K이며 초기조건 계약을 만족한다.
- 남은 residual은 float32 저장과 interval Joule heat 재구성에서 생기는 반올림 수준이다.
- 따라서 이후 FNO 예측 residual의 감소는 M2 지배식 위반 감소로 해석할 수 있다.
- 이 검사는 시뮬레이션 내부 일관성 검증이며 실제 용접 품질의 실측 검증은 아니다.

## 그림 읽기

1. `reference_residual_distribution.png`: 0 K 부근에 얼마나 집중되는지 본다.
2. `reference_residual_by_time.png`: 특정 시간 간격에서 재구성 오차가 커지는지 본다.
3. `reference_residual_worst_field.png`: 최악 오차가 경계·접촉면에 집중되는지 본다.
