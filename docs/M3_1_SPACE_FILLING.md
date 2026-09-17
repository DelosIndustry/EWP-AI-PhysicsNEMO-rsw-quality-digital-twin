# M3.1 확대 space-filling simulation 설계

## 왜 새 데이터 버전인가

M3 smoke train 64개는 `samples_per_group=4`이므로 실제 독립 공정 조건이 16개뿐이었다.
독립 uniform sampling 결과 validation 에너지가 train보다 낮게 치우쳤고, M5.2에서는 고온
OOD 과소예측이 확인됐다. 기존 E05 test에 맞춰 같은 manifest를 수정하지 않고
`RSW-SIM-V2`, seed `20260910`, generator `0.2.0`을 새로 만든다.

기존 `m3_dataset.yaml`은 계속 random-uniform generator `0.1.0`을 사용한다. 회귀 검사에서
기존 manifest ID `8a39a964572700bcf06c11dbebf3cde1b0d6430673ce17441d6384eaf51ae8be`와
파일 checksum이 그대로 재현됐다.

## 설계 크기

| split | sample | 독립 group | 역할 |
|---|---:|---:|---|
| train | 1,024 | 256 | IID 학습 |
| validation | 256 | 64 | 모델·threshold 선택 |
| test_iid | 256 | 64 | 동결 후 IID 보고 |
| test_process_ood | 256 | 64 | double-pulse·접촉조건 이동 |
| test_corner | 128 | 32 | 결합 극단 조건 |

총 1,920개다. grid와 출력 시간축은 기존 M3와 같은 `8×8×8`로 유지해 M4/M5 모델과 직접
비교할 수 있다.

## maximin Latin hypercube

연속 group parameter 8개를 각각 `n`개 구간으로 나누고, 각 축에서 모든 구간을 한 번씩
사용한다.

\[
x_{i,d}=l_d+(u_d-l_d)
\frac{\pi_d(i)+\epsilon_{i,d}}{n},
\qquad \epsilon_{i,d}\sim U(0,1)
\]

여기서 `π_d`는 축마다 다른 순열이다. 후보 12개 중 최근접 점 간 거리가 가장 큰 설계를
선택한다.

\[
X^*=\arg\max_X\min_{i\ne j}\|X_i-X_j\|_2
\]

초기온도, 냉각계수, 전류 jitter도 sample 수준의 별도 LHS를 쓴다. pulse shape은 group
수 차이가 최대 1이 되도록 균형 배분한다.

## solver-free preflight

다음 명령은 M1/M2 solver를 실행하지 않고 실제 생성에 사용될 parameter plan만 검사한다.

```bash
conda activate physicsnemo-study
python scripts/preflight_rsw_space_filling.py --overwrite
```

현재 결과:

- preflight ID: `50b939730b79d88ab42e0767140e9bd3a9bb58fc92644417adc6327fb7c28c1d`
- parameter plan SHA-256: `a52552ba74f16a4052e24679238bb7fad0c921f58931ba4b54fc476b9fdbd18c`
- 모든 연속변수의 LHS strata 점유율: 100%
- 모든 split pair의 group overlap: 0
- train rectangular/ramp groups: 128/128
- train joint high-energy groups: 9개, validation: 3개, test IID: 1개
- 비압축 field 예상량: 약 52.5 MiB
- 기존 M3 압축률 기반 NPZ 예상량: 약 6.2 MiB
- 3배 안전계수를 적용한 저장공간 gate: 통과

설계 확인용 contact energy proxy는 다음 식이다.

\[
E_{contact}^{proxy}=I^2t\frac{\rho_c}{\pi r_c^2}\,d_{pulse}
\]

이는 접촉부 Joule energy의 사전 순위 지표일 뿐 M2의 실제 energy 또는 온도 정답이 아니다.

## 전체 데이터 생성

사용자가 직접 다음을 실행한다.

```bash
conda activate physicsnemo-study
python scripts/generate_rsw_space_filling_data.py
```

생성 스크립트는 config hash뿐 아니라 parameter-plan hash까지 preflight와 다시 대조한다.
완료 후 다음을 확인한다.

1. 모든 split checksum과 group leakage 검사 통과
2. 실제 `Tmax`와 electrical energy 분포에서 train/validation 편향 감소
3. 에너지 수지와 Joule-terminal energy gap 통과
4. process/corner OOD가 별도 분포로 유지되는지 그림으로 확인
5. 비물리적 극고온이 나오면 학습하지 말고 물성·상변화 모델 범위를 먼저 재검토

용융온도는 아직 `null`이다. 사전 근거 없이 Mendeley category를 simulation PASS/FAIL로
변환하지 않는다.

## 시각화

preflight 산출물은 `artifacts/m3_1_design_preflight`에 생성된다.

- `marginal_coverage.png`: 8개 연속변수의 정규화 marginal coverage
- `joint_coverage.png`: 전류-접촉저항 결합 coverage와 energy proxy
- `energy_proxy.png`: split별 proxy 분포
- `split_budget.png`: sample/group/pulse 배분

전체 solver 실행 뒤에는 `artifacts/m3_1_dataset_audit`에 실제 온도·에너지 기반 그림이
별도로 생성된다.

## 전체 생성 후 판정

전체 생성은 완료됐고 manifest ID는
`2e43da1c058aef17a5500145fc92ea241b038457973adbcddb7162c1b39d2b98`다. checksum, group
leakage, tensor schema와 수치보존은 모두 통과했다. train-validation Tmax/energy KS도
`0.0771`/`0.0674`로 설정 기준 `0.10`을 통과했다.

다만 최고 Tmax가 `2004.811 K`이고 현재 M2에는 상변화·잠열·온도의존 물성이 없다. M3.2
물리 타당성 gate 결과 V2 전체범위 surrogate 학습은 차단했다. 상세 근거와 그림은
`docs/M3_2_PHYSICAL_VALIDITY.md`와 `artifacts/m3_2_physical_validity`를 따른다.
