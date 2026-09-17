# M7.2 Polito 공장 Fault 보조 track

## 목적

Polito 자동차 점용접 데이터의 upstream normalized 전압·전류·force 시계열로 설비 Fault를
보조 탐지한다. 이 label은 Mendeley의 Good/Bad/Explode와 의미가 다르므로 병합하지 않는다.
신호도 0–1 정규화 값이므로 V/A/N, 동저항 또는 Joule energy로 복원하지 않는다.

## M7.2a: test 봉인 개발

### 데이터·누수 계약

- train: 1,380행, Fault 55개, Car Body 20개
- validation: 300행, Fault 12개, Car Body 4개
- test: 296행, M7.2a에서는 파일 open/hash 모두 금지
- `sample_row_id` 중복 0, train-validation Car Body overlap 0
- model 입력에서 `Car Body`, 날짜, Welding Spot, row ID, target 제외

각 전압·전류·force에서 padding을 제외한 23개 통계량, 총 69개 feature를 만들었다. CNN은
세 신호와 세 validity mask를 6채널로 사용했다. Force 유효 길이가 0인 행은 train 166개,
validation 63개이며 보간하지 않고 length/mask 0으로 보존했다.

### 후보 모델

| 모델 | Validation PR-AUC | ROC-AUC | Fault recall | Fault precision | Brier |
|---|---:|---:|---:|---:|---:|
| Prior | 0.0400 | 0.5000 | 0.0000 | 0.0000 | 0.0384 |
| Balanced logistic | 0.2331 | 0.8429 | 0.6667 | 0.1600 | 0.1166 |
| Balanced Random Forest | **0.6104** | **0.8799** | 0.5000 | **0.6667** | **0.0298** |
| 1D CNN, epoch 14 | 0.2308 | 0.8041 | **0.8333** | 0.0901 | 0.2380 |

Fault prevalence가 4%이므로 전부 Normal로 예측해도 accuracy는 96%다. 따라서 accuracy는 선택
기준에서 제외하고 PR-AUC → Fault recall → Brier 순서로 Random Forest를 선택했다.

### 개발 triage

Validation에서 관측 Fault를 자동 NORMAL로 보내지 않는 경계와, 관측 Normal을 자동 FAULT로
보내지 않는 경계를 고정했다.

\[
\tau_{normal}=\min_{i:y_i=1}p_i=0.020198
\]

\[
\tau_{fault}=\operatorname{nextafter}\left(\max_{i:y_i=0}p_i,+\infty\right)
=0.705027
\]

- `P(Fault) < 0.020198`: NORMAL
- `P(Fault) >= 0.705027`: FAULT
- 그 사이: REVIEW

Validation 결과는 NORMAL 127, REVIEW 169, FAULT 4로 auto coverage 43.67%, review 56.33%다.
Fault 12개는 NORMAL 0, REVIEW 8, FAULT 4였다. 이는 validation 관측 오류 0일 뿐 미래 보장이
아니다.

선택 manifest ID는
`2ca70b27dc8420683d9914cf2882199c9b9cdff6b6fb8e79bf4a79545612b653`다.

## M7.2b: 동결 test 단 한 번 평가

M7.2a 선택 manifest, Random Forest pickle, feature, 두 임계값, M7.2b config/evaluator/runner를
SHA-256으로 먼저 검증했다. protocol ID는
`c33ff7a804748c90a747f1e9347977d6bf4c6063363525c2c6590fffce7b099c`다.

모델 fit, 모델 재선택, threshold 조정, validation 재참조를 금지한 뒤 test 296행을 열었다.

### 동결 분류 결과

| 지표 | Validation | Frozen test |
|---|---:|---:|
| PR-AUC | 0.6104 | 0.4157 |
| ROC-AUC | 0.8799 | 0.7994 |
| Fault recall @ 0.5 | 0.5000 | 0.3333 |
| Fault precision @ 0.5 | 0.6667 | 0.4000 |
| Brier | 0.0298 | 0.0383 |

Test confusion matrix는 실제/예측 순서 Normal, Fault 기준 다음과 같다.

\[
\begin{bmatrix}
278 & 6\\
8 & 4
\end{bmatrix}
\]

Fault 12개 중 4개를 검출했다. recall 95% Wilson 구간은 `[0.1381, 0.6094]`, precision 구간은
`[0.1682, 0.6873]`로 넓다. PR-AUC는 prevalence `0.0405`보다 높아 순위 신호는 존재하지만,
양산 경보기로 승인할 수준은 아니다.

### 동결 triage 결과

| 실제 label | NORMAL | REVIEW | FAULT |
|---|---:|---:|---:|
| Normal (284) | 131 | 153 | 0 |
| Fault (12) | 2 | 8 | 2 |

- Fault escape: `2/12 = 16.67%`
- Normal reject: `0/284 = 0%`
- Review rate: `161/296 = 54.39%`
- Auto coverage: `135/296 = 45.61%`
- Fault safe capture: `10/12 = 83.33%`, Wilson `[55.20%, 95.30%]`

Validation에서 0이던 false escape가 독립 test에서 2건 발생했다. 따라서 **재현성·무결성
검사는 통과했지만 성능 release는 실패**다. 이 test로 threshold를 다시 맞추지 않는다.

Car Body cluster bootstrap은 group이 네 개뿐이어서 매우 불안정하다. Fault escape 95% 구간은
`[0, 50%]`, review rate는 `[45.65%, 93.55%]`다. 새 공장·차체 holdout이 필요하다.

## 산출물

### M7.2a

- `metrics.json`, `selection_manifest.json`
- candidate/validation/CNN history/bootstrap/feature-importance CSV
- 동결 모델 4개와 PNG 7장
- 자동 생성 `INTERPRETATION.md`

### M7.2b

- `metrics.json`, `evaluation_receipt.json`
- test prediction/metrics/bootstrap/Wilson CSV
- PNG 7장
- 제목만 바로잡은 `06_test_triage_corrected.png`와 `presentation_amendment.json`

최종 evaluation receipt ID는
`0c115d373eb8e76a79d59817f548ac046ee03a3719ae1906cf3daf91eda382c5`다. 원본 receipt와 output
hash는 유지했고, 그림 제목 교정은 prediction·metric·threshold를 바꾸지 않은 별도 amendment다.

## 최종 판단

1. 통계 RF는 1D CNN보다 이 작은 불균형 데이터에서 강했다.
2. 정확도 95.27%는 Normal 다수 클래스 때문에 과장되므로 핵심 성과로 쓰지 않는다.
3. PR-AUC 0.4157은 무작위 0.0405보다 유의미한 순위 신호지만 Fault recall 0.3333은 부족하다.
4. triage는 Normal 오경보를 막았지만 Fault 2개를 자동 NORMAL로 보냈으므로 안전 gate 실패다.
5. test는 이미 소비됐고 이후 개선은 새 독립 holdout으로 검증해야 한다.
6. 이 결과는 생산기술 포트폴리오에서 “모델 성능 과장 대신 누수 방지·불확실성·fail-safe를
   검증한 사례”로 제시한다.
