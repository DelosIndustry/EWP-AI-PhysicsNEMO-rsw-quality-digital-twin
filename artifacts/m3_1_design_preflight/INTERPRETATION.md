# M3.1 space-filling 설계 preflight 해석

## 결론

- solver 실행 전 parameter plan 검증은 `True`다.
- train은 1024 samples, 256개의 독립 group 조건이다.
- 8개 연속 group parameter는 각 split에서 Latin-hypercube strata를 빠짐없이 한 번씩
  사용한다. 같은 범위 안에서 특정 구간이 우연히 비는 M3 smoke 문제를 줄인다.
- group ID의 split 간 교집합은 모두 0이다.
- 예상 비압축 field 크기는 52.5 MiB,
  기존 M3 압축률 기반 예상 NPZ 크기는 약
  6.2 MiB다.

## 그림 읽는 법

1. `marginal_coverage.png`: 0~1은 각 parameter의 설정 범위를 정규화한 값이다. 막대가
   decile 전반에 고르게 있어야 한다.
2. `joint_coverage.png`: 전류-접촉저항 조합과 `I²tρ/A` proxy를 함께 본다.
3. `energy_proxy.png`: split별 solver-free 에너지 proxy 범위다. 실제 온도나 품질 label이
   아니며 설계 사전 점검에만 쓴다.
4. `split_budget.png`: sample/group 수와 pulse shape 배분을 확인한다.

## 다음 실행

이 preflight는 온도장을 생성하지 않았다. 전체 M1/M2 simulation을 실행한 뒤에는 반드시
실제 `Tmax`, energy balance, parameter-output 분포 그림을 다시 검토해야 한다.
