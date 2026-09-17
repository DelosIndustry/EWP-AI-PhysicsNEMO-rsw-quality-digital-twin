# M0 PhysicsNeMo 모델 smoke 가이드

## 목적

실제 데이터를 만들기 전에 FNO만 가능한 프로젝트가 아닌지 확인한다. 무작위 tensor로
다음 계약만 검증한다.

- FNO3D: `B,C,T,R,Z -> B,1,T,R,Z`
- Transolver: `B,(T*R*Z),C + B,(T*R*Z),3 -> B,(T*R*Z),1`
- Transolver 출력을 다시 `B,1,T,R,Z`로 복원
- 두 모델의 loss backward와 finite gradient

## 실행

```bash
conda activate physicsnemo-study
python scripts/smoke_rsw_model_candidates.py
```

작은 GPU 실행을 명시하려면 다음처럼 실행한다.

```bash
python scripts/smoke_rsw_model_candidates.py \
  --device cuda \
  --time 4 \
  --radial 8 \
  --axial 8
```

## 결과 해석

- `output_shape`가 두 모델 모두 같아야 한다.
- `finite_gradients`가 모두 `true`여야 한다.
- parameter count나 loss 값으로 우수한 모델을 선택하지 않는다.
- 이 단계의 입력은 무작위이므로 물리적 정확도나 용접 품질을 의미하지 않는다.

현재 PhysicsNeMo 2.1.1에서는 Transolver의 3축 structured position helper 대신
`(t,r,z)` 좌표를 직접 전달한다. 최신 문서만 보고 `structured_shape=(T,R,Z)`로 바꾸지
않는다.
