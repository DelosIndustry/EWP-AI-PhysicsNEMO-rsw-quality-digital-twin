# M3 simulation dataset 해석

## 결론

- manifest ID: `8a39a964572700bcf06c11dbebf3cde1b0d6430673ce17441d6384eaf51ae8be`
- stack, schedule, contact group의 split 간 중복은 모두 0이다.
- NPZ 입력은 `[N, 10, T, R, Z]`, target은 `[N, 1, T, R, Z]`다.
- 현재 melt CTQ는 용융온도가 미설정되어 NaN으로 저장된다.

## 그림 읽는 법

1. `parameter_coverage.png`: IID와 OOD 범위가 의도대로 분리되는지 본다.
2. `output_distributions.png`: 설계된 입력 범위가 온도·에너지 범위를 어떻게 넓히는지 본다.
3. `train_correlations.png`: train 내부의 선형 연관만 확인하며 인과관계로 해석하지 않는다.
4. `corner_map.png`: 고전류·고접촉저항 corner sample이 실제로 존재하는지 본다.

## 주의

- 이 데이터는 공개 실측 데이터가 아니라 M1/M2 solver로 생성한 합성 field다.
- OOD는 실제 공장 규격 이탈이 아니라 학습 실험을 위해 설계한 상대 범위다.
- 절대 온도와 공정 품질의 연결은 외부 데이터 보정 전에는 주장하지 않는다.
