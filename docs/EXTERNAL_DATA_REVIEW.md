# 외부 점용접 데이터 검토

검토일: 2026-09-03

## 결론

점용접 실측 데이터는 존재하지만 내부 전체 온도장 `T(t,r,z)`은 제공하지 않는다. 따라서
PhysicsNeMo field 모델은 자체 전기-열 solver 데이터로 학습하고, 외부 자료는 CTQ·불량
검증에만 사용한다.

## 채택 후보

| 데이터 | 내용 | 적합한 역할 | 제한 |
|---|---|---|---|
| [Mendeley RSW Insights v3](https://data.mendeley.com/datasets/rwh8kjzdch/3) | 전류·시간·가압·재료, nugget, pull force, 열/가시광 이미지 | 첫 experimental CTQ adapter | 소형 수동 장비, 내부 field 없음 |
| [Polito automotive RSW](https://github.com/smartdatapolito/resistance_spot_welding_dataset) | 자동차 현장의 전압·전류·force 시계열과 fault label | factory time-series/OOD 별도 track | 물리 단위가 제거된 0~1 신호, 극심한 불균형 |

## Mendeley 데이터

- version 3, DOI `10.17632/rwh8kjzdch.3`, CC BY 4.0
- 495개 용접 sample
- 입력: pressure, weld time, electrode angle/force, current, thickness, material
- 출력: pull force, nugget diameter, `good/bad/expulsion`, 열화상·가시광 이미지
- 논문 보고 범위에는 weld time `200..1500 ms`, current 약 `640..5009 A` 등이 있지만,
  실제 archive를 읽어 단위와 결측치를 확인하기 전에는 simulation config 범위로 복사하지 않는다.
- 열화상은 반사 문제 때문에 용접 중 내부 온도가 아니라 용접 직후 냉각 단계의 표면을
  관측한다. field label로 쓰지 않는다.

원 데이터 설명과 실험 절차는 공개 논문에 있다.
[Data in Brief article](https://pmc.ncbi.nlm.nih.gov/articles/PMC11893339/)

## Polito 자동차 데이터

- 실제 자동차 산업에서 수집한 1,976개 sample
- 정상 1,897개, fault 79개
- 28개 car body와 158개 welding point
- 전압·전류·force 시계열을 0~1로 정규화하고 1,000 step으로 zero-pad
- 날짜 범위가 있어 car-body group test와 temporal test를 구성할 수 있음

이 자료는 절대 전류·전압 단위가 없어 electro-thermal solver의 직접 입력으로 사용하지
않는다. 원 저장소에서 명시적 license를 확인하기 전까지 원본 재배포와 저장소 commit도
하지 않는다.

## 사용 순서

1. M0.5에서 version/commit에 고정된 원본을 받고 checksum manifest를 만든다.
2. CSV schema·단위·결측·ID 중복을 읽기 전용으로 감사한다.
3. M1/M2 solver를 외부 field label 없이 검증한다.
4. simulation-only 모델 비교를 먼저 고정한다.
5. development subset에서 관측되지 않은 contact parameter 보정 방법을 고정한다.
6. 남겨둔 external test에서 nugget diameter와 class 결과를 한 번 평가한다.
7. Polito 자료는 별도 시계열 baseline과 OOD 연구로만 사용한다.

## 금지되는 주장

- 열화상으로 판재 내부 전체 온도장을 검증했다.
- 495개 실험 sample로 PhysicsNeMo field 모델을 직접 학습했다.
- 정규화된 자동차 시계열을 절대 단위 전류·전압으로 복원했다.
- 학습용 melt proxy가 실제 양산 nugget 판정을 대체한다.
