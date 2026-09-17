# 공개 데이터 저자에게 확인할 질문 (초안, 전송하지 않음)

1. `Data_RSW.csv`의 `Force (N)`와 Sensors 2025 Table 5의 `Electrode pressure (PSI)` 중 어느 단위와 quantity가 맞습니까?
2. HX711/load-cell calibration factor의 입력 질량 단위와 archived signal까지의 변환식을 제공할 수 있습니까?
3. load cell의 장착 위치, piston 면적, lever ratio, 마찰 보정 및 electrode-tip force 전달식을 제공할 수 있습니까?
4. 10 Hz record는 실제 timestamp와 동기화된 연속 waveform입니까, 아니면 weld 중 추출된 point sequence입니까?
5. 6000 A:5 A CT와 ACS712를 거친 archived current의 scaling, RMS/instantaneous 의미와 filter를 제공할 수 있습니까?

답변을 받기 전에는 N↔PSI, kgf↔N 또는 CT ratio를 raw 열에 자동 적용하지 않는다.
