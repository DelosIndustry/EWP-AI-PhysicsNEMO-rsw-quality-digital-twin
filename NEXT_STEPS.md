# 다음 진행 단계

## 현재 우선순위: M8 최종 포트폴리오 패키지 — 2026-09-18 마감

M7.2까지 구현과 동결 외부 평가를 완료했다. Polito frozen test PR-AUC는 `0.4157`로 prevalence
`0.0405`보다 높지만 Fault recall은 `0.3333`, triage false escape는 `16.67%`다. 따라서
재현성·누수 방지 성과는 확보했지만 production 성능 승인은 실패했다. Mendeley와 Polito test는
모두 소비됐으므로 이후 tuning에 다시 사용할 수 없다.

다음 순서로 진행한다.

1. 한 페이지 executive summary: 문제, 왜 PhysicsNeMo인지, 최종 결론과 production 차단 이유
2. 핵심 그림 6–8개 선별: solver 검증, FNO OOD, CTQ test, M7.1 gate, Polito frozen test
3. 현대자동차 생산기술 연결: 품질·설비 진단·가동률·디지털트윈·투자 의사결정
4. 재현 명령 한 페이지: 환경, 데이터 manifest, 주요 runner, 전체 test
5. 성공과 실패를 분리: 물리/재현성 성공, field calibration/자동 품질 승인 실패
6. Git untracked 파일을 검토해 프로젝트 단위 `.gitignore`와 첫 snapshot 준비
7. 3분 발표 스크립트와 예상 기술 질문·답변 작성

현재 검증 명령은 다음과 같다.

```bash
conda activate physicsnemo-study
python -m pytest tests/test_rsw_polito_fault_development.py \
  tests/test_rsw_polito_fault_frozen_test.py -q
```

M6.2와 M7.2b frozen test runner는 결과 디렉터리가 존재하면 재실행을 거부하는 것이 정상이다.
M3.2 전체범위 FNO 학습, automotive physical twin과 RSW-SIM-V3 차단 상태도 유지한다.


## M6 완료 결과

- Mendeley: 4,186 record → 495 weld, development/test 396/99, 충돌 weld 5개
- Polito: 1,976 rows, train/validation/test 1,380/300/296, fault 55/12/12
- 원본·변환 checksum 및 manifest ID 일치
- `Sample ID`와 `Car Body` group leakage 0
- 평가 JSON, 해석 문서와 PNG 4장 생성

```bash
conda activate physicsnemo-study
python scripts/prepare_rsw_external_data.py
python -m pytest tests/test_rsw_external_adapter.py -q
```

## M0.5 외부 데이터 감사

먼저 다운로드된 실측 데이터의 독립 sample 단위, 중복 key, class 불균형과 시계열 padding을
확정한다.

```bash
conda activate physicsnemo-study
python scripts/audit_rsw_external_data.py
```

통과 조건은 다음과 같다.

- Mendeley split 단위를 CSV 행이 아니라 `Sample ID`로 고정
- Polito 네 파일의 row metadata 순서 일치
- Polito join용 `sample_row_id` 도입 결정
- `Car Body` group split과 fault-aware 평가 지표 결정
- 감사 JSON, PNG 네 장과 `INTERPRETATION.md` 생성

## M1 축대칭 전기 solver

다음 구현은 ML 모델이 아니라 reference field를 만드는 전기전도 solver다.

1. `r-z` cell-centered finite-volume grid를 정의한다.
2. 보존형 flux로 `div(sigma grad(phi)) = 0`을 이산화한다.
3. 전극 경계에 전류 또는 전위를 적용하고 `r=0` 축 대칭을 처리한다.
4. `J = -sigma grad(phi)`와 `q_joule = J dot J / sigma`를 계산한다.
5. 균일 도체 해석해, 전류 보존, Joule heat 비음수, 전류 제곱 scaling을 테스트한다.
6. grid refinement 결과를 수치와 그림으로 동시에 저장한다.

M1 평가 그림은 다음 네 가지를 기본으로 한다.

- 해석해와 중심선 전위의 겹침 line plot
- `phi(r,z)` contour
- `|J(r,z)|`와 `q_joule(r,z)` contour
- grid spacing 대비 오차의 log-log convergence plot

통과 기준은 해석해 오차 감소, 유입·유출 전류 보존, 모든 cell에서 `q_joule >= 0`,
입력 전류를 두 배로 했을 때 Joule heat가 약 네 배가 되는 것이다.

## M2 이후

M1이 통과하면 과도 열전도와 순차 연성을 추가한다. 그 뒤에만 simulation Dataset을 만들고
CNN/FNO3D/Transolver를 동일 split에서 비교한다. 각 단계는
`docs/VISUALIZATION_GUIDE.md`의 JSON·PNG·해석문서 계약을 따른다.
