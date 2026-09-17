# M7.2a 결과 해석 — Polito 개발 전용 Fault 보조 모델

## 한 줄 결론

선택 모델은 `random_forest_balanced`이고 validation PR-AUC는
`0.6104`다. 하지만 validation은 Car Body 4개와 Fault 12개뿐이고
test 296행은 아직 열지 않았으므로, 이 수치는 공장 성능 확정값이 아니다.

## 왜 accuracy보다 PR-AUC인가

validation 300행 중 Fault는 12개(4%)다. 모든 행을 Normal로 예측해도 accuracy는 96%이므로
accuracy는 거의 쓸모가 없다. PR-AUC는 Fault를 놓치지 않는 recall과 경보의 precision을 함께
보며, 무작위 기준은 Fault prevalence인 약 0.04다.

- PR-AUC: 0.6104
- ROC-AUC: 0.8799
- 0.5 threshold Fault recall: 0.5000
- 0.5 threshold Fault precision: 0.6667
- Brier score: 0.0298

CNN 학습 장치는 `cuda`였다. 모델 선택은 PR-AUC, 0.5 Fault recall, Brier 순서이며
accuracy는 선택 기준이 아니다.

Force 유효 길이가 0인 행은 train
`166`개, validation
`63`개다. 값을 만들어 채우지 않고 길이 0과
mask 0으로 보존했다. validation에서 force 결측률이 더 높으므로 분포 이동 가능성을 함께
봐야 한다.

## NORMAL / REVIEW / FAULT

validation에서 관측된 Fault를 자동 NORMAL로 보내지 않는 최대 NORMAL 영역과, 관측된 Normal을
자동 FAULT로 보내지 않는 최대 FAULT 영역을 만들었다.

- NORMAL 조건: `P(Fault) < 0.020198`
- FAULT 조건: `P(Fault) >= 0.705027`
- 두 조건 사이 또는 겹치는 영역: REVIEW

결과는 fault escape `0/12`,
normal reject `0/288`, review rate
`0.563`, auto coverage `0.437`다. 관측 오류 0은 미래
오류 0을 보장하지 않는다.

Car Body cluster bootstrap에서 PR-AUC 95% 구간은
`[0.6000, 1.0000]`다. 그룹이 4개뿐이라
구간 자체도 불안정하므로 점추정치보다 새로운 공장·차체 group 평가가 더 중요하다.

## 그림 읽는 순서

1. `01_split_distribution.png`: class 불균형과 Car Body별 Fault 비율
2. `02_validation_signal_profiles.png`: Normal/Fault 평균 신호 형상
3. `03_validation_pr_curves.png`: 낮은 prevalence에서 가장 중요한 모델 비교
4. `04_validation_roc_curves.png`: 보조 순위 지표
5. `05_validation_calibration.png`: 예측 확률과 실제 빈도의 일치
6. `06_validation_triage.png`: 실제 Normal/Fault가 세 판정으로 간 위치
7. `07_rf_feature_importance.png`: RF가 사용한 통계 feature; 인과 해석 금지

## 제한

1. 전압·전류·force는 upstream 0–1 정규화 값이라 V/A/N 또는 Joule energy로 해석할 수 없다.
2. Car Body, 날짜, Welding Spot은 shortcut을 막기 위해 model feature에서 제외했다.
3. Mendeley Good/Bad/Explode와 Polito Fault는 의미가 달라 병합하지 않는다.
4. 현재 모델은 설비 이상 보조 분류기이며 PhysicsNeMo 온도장 surrogate 검증이 아니다.
5. 비용 20:5:1과 validation 임계값은 학습용이며 자동차 생산 요구사항이 아니다.
6. test는 선택 manifest와 모델이 동결된 다음 M7.2b에서 단 한 번 평가한다.
