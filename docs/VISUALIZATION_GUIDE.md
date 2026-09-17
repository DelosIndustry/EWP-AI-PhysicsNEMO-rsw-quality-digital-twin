# 평가 시각화 규칙

## 원칙

앞으로 모든 정량 평가에는 숫자만 있는 console log 대신 다음 세 산출물을 함께 남긴다.

```text
artifacts/<experiment_id>/
├── metrics.json 또는 audit_summary.json
├── figures/*.png 또는 해석 가능한 PNG
└── INTERPRETATION.md
```

- 그림은 train 결과가 아니라 validation/test split과 reference를 명시한다.
- 축 이름에 단위를 쓰고, normalized 값은 반드시 `Normalized`로 표시한다.
- PASS/FAIL 선은 출처가 정해진 뒤에만 그린다.
- 그림에 사용한 sample ID와 split을 기록해 좋은 사례만 골라 보여주지 않는다.
- 평균 그림과 실패 사례 그림을 함께 저장한다.
- 색만으로 구분하지 않고 label, line style 또는 marker를 함께 사용한다.

## 단계별 필수 그림

| 단계 | 필수 그림 | 해석 질문 |
|---|---|---|
| M0.5 외부 감사 | class balance, group size, signal length, 대표 시계열 | 독립 sample 단위와 불균형은 무엇인가? |
| M1 전기 solver | 해석해-수치해 line, 전위/전류/Joule heat contour, 수렴곡선 | 전류가 보존되고 오차가 격자와 함께 줄어드는가? |
| M2 열 solver | 중심온도 이력, 온도 contour, energy balance, 시간수렴 | 열원이 없을 때 보존되고 에너지 오차가 작은가? |
| M3 Dataset | parameter coverage, split별 분포, correlation, corner map | train/test 누수 없이 필요한 공정범위를 덮는가? |
| M4/M5 모델 | reference/prediction/error panel, CTQ parity, error histogram | 어디서 틀리고 단순 기준선보다 나은가? |
| M6 외부 검증 | proxy-measurement scatter, residual by condition | sim-to-real gap이 특정 재료·조건에 집중되는가? |
| M7 품질 gate | confusion matrix, coverage-error, review trade-off | false escape를 줄이면서 자동판정 범위를 확보하는가? |
| M8 mesh 비교 | accuracy-latency-memory Pareto | 복잡한 mesh의 비용이 정확도 개선으로 보상되는가? |

## 첫 실행

외부 원본을 내려받은 뒤 다음 명령으로 감사 그림 네 장과 해석 문서를 생성한다.

```bash
python scripts/audit_rsw_external_data.py
```

결과는 `projects/rsw_quality_digital_twin/artifacts/m0_5_external_audit/`에 생성된다.

### 그림 읽는 순서

1. `mendeley_overview.png`: 행 기준이 아니라 weld ID 기준 category 균형을 본다.
2. `mendeley_example_traces.png`: 같은 Sample ID 안에서 current/force가 반복 측정임을 본다.
3. `polito_overview.png`: 0/1 label 불균형, car body group, padding과 날짜 drift를 본다.
4. `polito_signal_examples.png`: 정규화된 전압·전류·force 형상 차이를 확인한다.

그림은 인과관계를 증명하지 않는다. 예를 들어 current와 nugget의 산점도 상관은 접촉,
두께, 재료와 통전시간의 영향을 통제한 결과가 아니다.

