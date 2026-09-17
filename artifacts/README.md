# 점용접 실험 산출물

checkpoint, 원시 log, 대형 VTK/NPZ/HDF5/Zarr 파일은 Git에 commit하지 않는다. 최종
보고에 필요한 작은 JSON/CSV와 대표 그림만 출처 manifest와 함께 남긴다.

권장 구조:

```text
artifacts/
├── m0_model_smoke/
├── m1_electrical_solver/
├── m2_coupled_solver/
├── m4_model_benchmark/
├── m6_external_validation/
└── m7_quality_gate/
```

`m6_external_validation/`에는 adapter의 split·충돌·row alignment 검증 JSON과 해석 가능한
PNG만 둔다. 원본 및 처리된 외부 데이터는 `data/` 아래의 Git 제외 경로에 유지한다.
