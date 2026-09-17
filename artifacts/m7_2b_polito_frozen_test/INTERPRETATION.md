# M7.2b 결과 해석 — Polito 동결 test

## 한 줄 결론

M7.2a에서 선택한 Random Forest, 69개 feature와 두 판정 임계값을 바꾸지 않고 Polito test
296행을 한 번 평가했다. test PR-AUC는 `0.4157`, ROC-AUC는
`0.7994`다. 모델 재학습·재선택·threshold 조정은 수행하지 않았다.

## 0.5 분류 결과

- Fault: 12개
- Fault recall: 0.3333
- Fault precision: 0.4000
- Balanced accuracy: 0.6561
- Brier score: 0.0383

Fault recall의 95% Wilson 구간은
`[0.1381,
0.6094]`다. Fault가 12개뿐이어서 점추정치가 좋아도
불확실성이 크다.

Validation PR-AUC `0.6104`와 test PR-AUC
`0.4157`의 차이는 새 Car Body group에서 일반화가 얼마나 유지됐는지 보여준다.
두 split 모두 Fault 12개이므로 작은 건수 차이를 과도하게 해석하면 안 된다.

## 동결 NORMAL / REVIEW / FAULT 결과

- NORMAL: 133개
- REVIEW: 161개
- FAULT: 2개
- Fault escape: 2/12
- Normal reject: 0/284
- Review rate: 0.5439
- Auto coverage: 0.4561

이 임계값은 validation에서 정했고 test를 보고 바꾸지 않았다. test 결과가 나쁘더라도 현재
test에 맞춰 임계값을 조정하면 안 된다. 개선에는 새로운 독립 holdout이 필요하다.

## 제한

1. test Car Body는 4개뿐이며 cluster bootstrap 구간도 안정적 모집단 구간이 아니다.
2. 신호는 upstream normalized 값이므로 V/A/N, 저항 또는 Joule energy로 해석할 수 없다.
3. 이 모델은 Polito Fault 보조 분류기이며 점용접 CTQ나 PhysicsNeMo 온도장 모델이 아니다.
4. Mendeley category와 Polito Fault label은 계속 분리한다.
5. 자동 생산 품질 승인과 SI physics/physical-twin 주장은 계속 차단한다.
