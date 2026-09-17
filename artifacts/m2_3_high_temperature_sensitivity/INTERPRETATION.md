# M2.3 고온 벌크 전기저항 민감도 결과 해석

## 결론

- 수치 검증 9/9개가 통과했다.
- 300–1000 K와 1500–1800 K 직접 자료는 모든 scenario에서 그대로 유지했다.
- 1000–1500 K에는 직접 점을 만들지 않고 세 endpoint-preserving 가정만 비교했다.
- V3 준비 상태는 `blocked_grade_and_contact_calibration`다.

## 민감도

- 최고온도 scenario 범위: 0.742 K
- terminal energy 범위: 9.191 J
- 최대 액상분율 범위: 0.025583
- mushy diameter proxy 범위: 0.000 mm

`early_rise`가 gap 초반부터 큰 저항을 주므로 고정 전류에서 가장 많은 열을 넣고,
`late_rise`는 그 반대다. 세 결과의 차이는 측정되지 않은 bridge 형상만의 영향이다.

## 그림 읽는 법

1. `resistivity_evidence_and_bridges.png`: 점은 직접 자료, 회색 구간의 선은 가정이다.
2. `sensitivity_time_histories.png`: 온도·전압 곡선이 언제 벌어지는지 본다.
3. `sensitivity_summary.png`: 최종 품질 proxy의 scenario spread를 비교한다.
4. `energy_conservation.png`: terminal과 Joule energy의 일치 및 열수지를 본다.
5. `peak_field_comparison.png`: 같은 색 범위에서 공간장 차이를 비교한다.

## 남은 차단 조건

- 두 자료는 순철/electrolytic iron이며 자동차용 목표 강종 보정이 아니다.
- 설정의 300 K reference conductivity는 절대 벌크값이 아니라 기존 유효 scale이다.
- 판재–판재 및 전극–판재 접촉저항의 온도·가압력 의존성이 아직 분리되지 않았다.
- 따라서 bulk 온도범위는 연결했지만 RSW-SIM-V3 생성은 아직 승인하지 않는다.
