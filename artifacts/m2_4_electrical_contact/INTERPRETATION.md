# M2.4 명시적 전기 접촉저항 결과 해석

## 핵심 결론

- 자동 검증 12/12개가 통과했다.
- 접촉저항이 0이면 기존 M1의 전위와 전류를 재현한다.
- 새 발열장은 각 유한체적 저항의 I²R을 직접 분배하므로 별도 정규화가 없다.
- nominal 접촉저항 에너지 몫: 84.20%.
- nominal: Tmax 1288.80 K, 액상분율 0.0000.
- V3 준비 상태는 `blocked_contact_calibration_and_force_temperature_laws`다.

## 수식

축 방향 face 하나의 저항은 다음처럼 세 항을 직렬로 더했다.

`R_face = [(dz/2)/sigma_lower + R''_contact + (dz/2)/sigma_upper] / A_face`

접촉열 `P_contact = I_face² R_contact`는 양쪽 판재 셀에 1/2씩 분배했다.
전극-판재 접촉열 중 설정 비율만 판재에 넣고 나머지는 전극 흡수 에너지로 기록했다.
따라서 `E_terminal = E_sheet_source + E_electrode_absorbed`가 닫혀야 한다.

## 그림 읽는 법

1. `contact_resistance_profiles.png`: 반경별 R'' 가정과 접촉 반경을 확인한다.
2. `power_decomposition.png`: 벌크/판재-판재/전극-판재 에너지 몫을 비교한다.
3. `contact_sensitivity_histories.png`: 같은 전류의 온도·전압을 비교한다.
4. `energy_ledger.png`: 전기 및 열 에너지 보존 오차를 확인한다.
5. `nominal_heat_and_temperature_fields.png`: 접촉열이 계면 주변 셀에 놓이는지 본다.

## 아직 보정값으로 해석하면 안 되는 이유

- nominal R''는 기존 유효 전기저항을 벌크와 접촉으로 분해한 합성 초기값이다.
- low/nominal/high는 구현 민감도 시험이며 실측 접촉저항의 신뢰구간이 아니다.
- 가압력, 온도, 표면 거칠기, 도금에 따른 접촉저항 법칙은 아직 없다.
- 그러므로 M2.4는 물리 API와 에너지 장부 완성 단계이며 V3 calibration 완료가 아니다.
