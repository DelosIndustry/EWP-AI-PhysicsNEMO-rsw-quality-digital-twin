# M5.1 physics-loss ablation 해석

Validation Tmax MAE 최소 variant는 `data_only`이고, thermal residual 최소 variant는 `plus_thermal_residual`이다.

최종 선택은 64-sample overfit gate를 통과한 variant에 한정한다. 그림은 training → accuracy → physics constraints → worst fields 순서로 읽는다. 이 단계는 validation simulation 비교이며 test/OOD와 실측 품질 결론은 아직 봉인한다.
