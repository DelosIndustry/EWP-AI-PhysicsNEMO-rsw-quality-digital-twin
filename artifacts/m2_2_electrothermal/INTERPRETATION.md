# M2.2 온도의존 전기-열 약연성 결과 해석

## 결론

- 수치 검증 15/15개가 통과했다.
- 각 시간구간 시작 온도로 전기전도도를 갱신한 뒤 고정 전류 전기장을 다시 푼다.
- 고정 전류에서는 저항 증가가 필요한 전압과 Joule power를 함께 증가시킨다.
- V3 데이터 재생성 준비 상태: `blocked_electrical_property_temperature_range`.

## 핵심 수치

- 온도의존 case 최고온도: 634.869 K
- frozen case 최고온도: 575.021 K
- feedback 온도 증가량: 59.848 K
- 최종 terminal energy: 77.506819 J
- 최대 terminal-source gap: 1.726e-16
- 최대 thermal balance error: 5.177e-15

## 그림 읽는 법

1. `electrical_property_coverage.png`: 회색 구간은 전기 물성표가 없어 사용 금지된 온도다.
2. `feedback_history.png`: 실선과 frozen 점선의 벌어짐이 전기-열 feedback 효과다.
3. `energy_accounting.png`: 세 에너지 곡선은 겹치고 우측 오차는 허용치 아래여야 한다.
4. `field_snapshots.png`: 통전 말기의 온도, 저항비, Joule source 공간분포를 함께 본다.

## 제한과 다음 gate

- NIST electrolytic-iron 곡선의 상대 온도의존성만 사용했으며 저탄소강 절대물성 보정이 아니다.
- 옮긴 표는 300-1000 K뿐이라 1768-1797 K 상변화 영역을 덮지 못한다.
- 따라서 현재 solver 검증은 PASS지만 이 물성으로 V3 용융 데이터셋을 만들면 안 된다.
- 다음 단계는 강종별 고온 전기저항/접촉저항 자료 확보 또는 범위가 명시된 민감도 설계다.
