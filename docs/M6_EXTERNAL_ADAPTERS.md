# M6 외부 실측 데이터 adapter

## 목적

M6는 외부 데이터를 FNO 온도장 정답에 억지로 결합하는 단계가 아니다. 두 공개 데이터의
실제 관측 단위와 label 의미를 보존한 별도 데이터 계약을 만드는 단계다.

| manifest | 데이터 종류 | 독립 sample | 허용된 정답 | 금지된 사용 |
|---|---|---|---|---|
| `m6_mendeley_ctq_manifest.json` | `experimental_ctq` | `sample_id` | nugget 직경, pull force, category | 내부 온도장 정답 |
| `m6_polito_factory_manifest.json` | `factory_timeseries` | `sample_row_id` | `Fault` | SI 단위 복원, 내부 온도장 정답 |

## 실행

저장소 루트에서 다음을 실행한다.

```bash
conda activate physicsnemo-study
python scripts/prepare_rsw_external_data.py
```

같은 출력이 이미 있으면 실수로 덮어쓰지 않는다. 같은 config와 원본으로 재현성을 다시
확인하려는 경우에만 `--overwrite`를 붙인다.

검증 코드는 다음과 같다.

```bash
python -m pytest tests/test_rsw_external_adapter.py \
  tests/test_rsw_audit.py tests/test_rsw_external_data.py -q
ruff check src/physicsnemo_study/data/rsw_external_adapter.py \
  src/physicsnemo_study/data/rsw_external_report.py \
  scripts/prepare_rsw_external_data.py tests/test_rsw_external_adapter.py
```

## Mendeley 변환

원본 4,186행에는 같은 용접을 반복 기록한 행이 있다. 따라서 `Sample ID`가 같은 행은 항상
같은 split에 둔다. category별 test 수는 다음과 같이 결정한다.

\[
n_{test,c}=\operatorname{clip}\!\left(
\operatorname{round}(0.2n_c),1,n_c-1
\right)
\]

seed 42 결과는 development 396개, test 99개 weld다. Good/Bad/Explode는 각각
development `354/17/25`, test `89/4/6`이다.

정적 scalar가 한 `Sample ID` 안에서 여러 값을 가지면 첫 행을 임의 선택하지 않는다.

\[
x_{resolved}=\begin{cases}
x_1,& |\operatorname{unique}(x)|=1\\
\mathrm{NaN},& |\operatorname{unique}(x)|>1
\end{cases}
\]

동시에 `min`, `max`, `unique_count`, `is_conflicted`를 저장한다. 실제 원본에서는 5개 weld가
충돌하며, field-weld pair로는 두께 A 2개, 두께 B 2개, pull test 3개로 총 7개다.

## Polito 변환

metadata key가 중복되므로 `(Car Body, Welding Spot, Date)`로 merge하지 않는다. 네 CSV의
검증된 원본 행 순서에 `sample_row_id=0,...,1975`를 먼저 부여하고 전압·전류·가압력 배열을
같은 행 번호로 결합한다.

고정 split은 다음과 같다.

| split | Car Body 수 | sample | normal | fault |
|---|---:|---:|---:|---:|
| train | 20 | 1,380 | 1,325 | 55 |
| validation | 4 | 300 | 288 | 12 |
| test | 4 | 296 | 284 | 12 |

세 split의 fault 비율은 약 4%로 비슷하지만, 이는 class imbalance가 사라졌다는 뜻이 아니다.
후속 분류 평가는 accuracy 단독 사용을 금지하고 PR-AUC, fault recall, false-negative rate를
함께 보고한다.

각 신호의 유효 길이 `L_i`는 마지막 non-zero 위치로 계산하고 padding mask는 다음과 같다.

\[
M_{i,t}=\mathbb{1}[t<L_i]
\]

공개 신호는 0~1 정규화 값이다. 원래 scaling 정보가 없으므로 A, V, N 단위로 역변환하지
않는다.

## 산출물과 그림

- 처리 데이터: `data/processed_external/mendeley_ctq`, `polito_factory`
- manifest: `data/manifests/m6_mendeley_ctq_manifest.json`,
  `m6_polito_factory_manifest.json`
- 평가: `artifacts/m6_external_validation/m6_results.json`
- 해석: `artifacts/m6_external_validation/INTERPRETATION.md`
- 그림: split/CTQ, conflict, Car Body split, test signal 예시 총 4장

처리 데이터와 Polito 원본은 Git에 commit하지 않는다. 특히 Polito 원본 저장소에는 명시적
license가 없으므로 로컬 분석에만 사용하고 재배포하지 않는다.
