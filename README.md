# PhysicsNeMo 점용접 품질 디지털 트윈

## 한 문장 목표

차체 저항 점용접의 공정 조건과 재료·접촉 조건으로부터 시간에 따른 전기·온도장을
빠르게 근사하고, 용융부 직경과 과열 위험을 불확실성까지 고려해 판정하는 재현 가능한
PhysicsNeMo 프로젝트를 만든다.

> 이 프로젝트는 공개 실험 데이터와 공개 가능한 자체 시뮬레이션만 사용한다. 초기
> 물성·접촉 모델과 품질 한계값은 학습용 가정이며 현대자동차 또는 다른 완성차 업체의
> 실제 공정 사양을 재현하지 않는다. 모델 판정은 실제 용접 검사나 CAE 승인을 대체하지
> 않는다.

## 프로젝트 질문

1. PhysicsNeMo 모델은 전류 이력과 판재·접촉 조건에서 전체 온도 이력을 얼마나 빠르고
   정확하게 근사하는가?
2. 고정 격자의 FNO와 좌표 기반 Transolver 중 용융 경계와 시간 이력을 더 잘 예측하는
   모델은 무엇인가?
3. 데이터 loss에 전기·열 방정식 residual을 더하면 적은 데이터와 OOD 조건에서 도움이
   되는가?
4. 공개 실험의 nugget 직경·인장강도·불량 label로 시뮬레이션 기반 CTQ를 어디까지
   외부 검증할 수 있는가?
5. 불확실하거나 학습 범위를 벗어난 용접 조건을 자동 합격시키지 않고 `REVIEW`로 보낼
   수 있는가?

## MVP 범위

- 두 판재가 겹친 `r-z` 축대칭 2차원 영역
- 준정상 전기전도와 과도 열전도의 순차 연성
- 전극은 첫 단계에서 고체 mesh가 아니라 전류·냉각 경계조건으로 단순화
- 접촉 전기저항과 열접촉 전도도는 얇은 유효 계면으로 표현
- 자체 solver가 생성한 `T(t,r,z)` 학습 데이터
- 같은 시공간 tensor를 사용하는 PhysicsNeMo 3D FNO와 Transolver 비교
- 용융부 직경 proxy, 최고 온도, 냉각속도와 에너지 수지 평가
- 공개 실험·자동차 생산 시계열은 별도 manifest와 별도 결과표로 외부 평가

## MVP에서 하지 않는 것

- 전극 가압에 의한 탄소성 접촉 변형 직접 해석
- 용융 금속 유동, splash 또는 expulsion의 직접 해석
- 온도 의존 상변태와 금속조직의 완전한 재현
- 실제 공장 PLC의 폐루프 제어
- 공개 실측과 시뮬레이션 field를 같은 정답처럼 병합
- 첫 기준선 전에 3D 실차 형상, DDP 또는 대규모 탐색

## 생산기술 연구

| 생산기술 업무 | 프로젝트 산출물 |
|---|---|
| 차체 신차 공법·구조 검토 | 판재 stack과 용접 recipe의 what-if 비교 |
| 설비 사양 검토 | 전류·통전시간·접촉·냉각 조건의 안전 운전영역 |
| 품질 확보 | nugget proxy, 과열 위험, false escape와 `REVIEW` 정책 |
| 생산 데이터 활용 | 전류·전압·가압력 시계열과 CAE provenance의 분리·결합 |
| 스마트팩토리 | 빠른 surrogate, OOD 감지, 정밀 해석 fallback |
| 에너지·생산성 | 품질 제약 아래 에너지와 cycle time 비교 |

## 문서

- [문제와 물리 범위](docs/PROJECT_SPEC.md)
- [tensor·split·manifest 명세](docs/DATA_SPEC.md)
- [PhysicsNeMo 모델 비교와 선택](docs/MODEL_REVIEW.md)
- [외부 실험 데이터 검토](docs/EXTERNAL_DATA_REVIEW.md)
- [단계별 실험 계획](docs/EXPERIMENT_PLAN.md)
- [구현 및 검증 상태](docs/IMPLEMENTATION_STATUS.md)
- [M0 모델 smoke 실행 가이드](docs/M0_MODEL_SMOKE_GUIDE.md)
- [M2.1 enthalpy 상변화 solver](docs/M2_1_ENTHALPY_SOLVER.md)
- [M2.2 온도의존 전기–열 약연성](docs/M2_2_ELECTROTHERMAL_COUPLING.md)
- [M2.3 고온 벌크저항 민감도](docs/M2_3_HIGH_TEMPERATURE_SENSITIVITY.md)
- [M2.4 명시적 전기 접촉저항](docs/M2_4_ELECTRICAL_CONTACT_RESISTANCE.md)
- [M2.5 상태의존 접촉법칙·식별성](docs/M2_5_CONTACT_LAW_IDENTIFIABILITY.md)
- [M2.6 목표 stack·bounded preflight](docs/M2_6_TARGET_STACK_BOUNDED_PREFLIGHT.md)
- [M2.7 가압력 단위·보정 가능성 감사](docs/M2_7_CALIBRATION_FEASIBILITY.md)
- [M3.1 확대 space-filling 설계](docs/M3_1_SPACE_FILLING.md)
- [M3.2 simulation 물리 타당성 gate](docs/M3_2_PHYSICAL_VALIDITY.md)
- [M6 외부 실측 adapter](docs/M6_EXTERNAL_ADAPTERS.md)
- [M6.1 development-only 외부 CTQ 기준선](docs/M6_1_EXTERNAL_CTQ_BASELINE.md)
- [M6.2 동결 외부 test 평가](docs/M6_2_FROZEN_EXTERNAL_TEST.md)
- [M7.1 개발 전용 품질 gate·OOD feasibility](docs/M7_1_DEVELOPMENT_GATE.md)
- [M7.2 Polito 공장 Fault 보조 track](docs/M7_2_POLITO_FAULT_TRACK.md)

## 마일스톤

| 단계 | 결과물 | 통과 기준 |
|---|---|---|
| M0 | 문제·데이터·모델 계약 | FNO3D와 Transolver shape/gradient smoke 통과 |
| M1 | 축대칭 전기 solver | 해석해, 전류 보존과 Joule heat test 통과 |
| M2 | 과도 열 solver와 연성 | 무발열·에너지 수지·격자/시간 수렴 test 통과 |
| M2.1 | enthalpy 상변화 solver | 해석해, 잠열 0 회귀, 상분율·비선형·에너지 gate 통과 |
| M2.2 | 온도의존 전기–열 약연성 | frozen 회귀, 전류·교차에너지 gate 통과; 용융범위 물성은 BLOCK |
| M2.3 | 고온 벌크저항 민감도 | 3 bridge·상변화·에너지 gate 통과; 접촉/강종 보정 BLOCK |
| M2.4 | 명시적 전기 접촉저항 | 해석해·회귀·접촉열·에너지 gate 통과; 실측 보정 BLOCK |
| M2.5 | 상태의존 접촉법칙·식별성 | T/F law·rank 1·외부 관측성 통과; target stack BLOCK |
| M2.6 | 목표 stack·bounded preflight | 11 gates PASS; stack 근거 확보, force overlap·접촉 보정 BLOCK |
| M2.7 | 단위·sensor calibration feasibility | 16 gates PASS; lab CTQ 허용, physical twin·V3 BLOCK |
| M3 | 결정론적 시뮬레이션 Dataset | 같은 seed/config에서 같은 manifest 생성 |
| M4 | CNN/FNO/Transolver 기준선 | 동일 split·정규화·선택 metric 비교 완료 |
| M5 | physics residual | data-only 대비 PINO ablation 완료 |
| M6 | 외부 실험 adapter | simulation과 experimental 결과를 분리해 보고 |
| M6.1 | 외부 CTQ 기준선 | force 제외, development-only 선택·model hash 동결·PNG 완료 |
| M6.2 | 동결 외부 test | 재학습 없이 bootstrap·class recall·receipt 보고 완료 |
| M7.1 | 개발 전용 품질 gate와 OOD | nested CV, false escape/review, OOD flag rate 산출 |
| M7.2 | Polito 설비 이상 보조 track | 완료; frozen test PR-AUC 0.4157, 성능 release 실패 |
| M8 | mesh·보고서 확장 | MeshGraphNet 타당성 또는 최종 실패 분석 문서화 |

## 지금 실행할 명령

저장소 루트와 `physicsnemo-study` Conda 환경에서 실행한다.

```bash
conda activate physicsnemo-study
python -m pytest tests/test_rsw_polito_fault_development.py \
  tests/test_rsw_polito_fault_frozen_test.py -q
```

M6.2 Mendeley test와 M7.2b Polito test는 이미 소비됐다. 두 test를 사용한 재학습·모델 재선택·
threshold 조정은 금지한다. 다음은 새 실험을 늘리기보다 M8 최종 포트폴리오와 재현 실행 안내를
완성하는 단계다.
