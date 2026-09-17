# M2.6 해석

## 결론

- 수치·데이터 계약 gate: `11/11 PASS`
- V3 생성 준비: `False`
- target stack: `AISI1010_0p63x2_surfaceUnknown_CuAlloyTip3p175`
- development stack-compatible weld: `359` / `396`
- 기존 contact-law force 범위와 겹치는 weld: `0`

판재 강종과 약 0.63 mm 두께에는 공개 실험 근거가 있다. 하지만 도금/표면 상태, 정확한 전극 합금, 실제 전기 접촉반경과 계면저항은 관측되지 않았다. 따라서 이번 결과는 보정 완료가 아니라 안전한 불확실성 경계를 만든 것이다.

## screening DOE

- baseline Tmax: `1327.954 K`
- baseline terminal energy: `110.919 J`
- 가장 큰 OAT Tmax span: `faying_resistance_scale` = `899.705 K`

OAT span은 한 변수만 low/high로 바꾼 수치 민감도다. 실제 확률분포, 변수 상호작용 또는 실험적으로 보정된 중요도를 뜻하지 않는다.

## V3 차단 원인

- unknown: `effective_electrical_contact_radius, electrode_alloy_grade, surface_coating`
- proxy/assumption parameters: `bulk_resistivity_scale, electrode_force_exponent, electrode_resistance_scale, faying_force_exponent, faying_resistance_scale, latent_heat_scale, screening_peak_current_a, volumetric_heat_capacity_scale`
- nuisance parameters: `effective_contact_radius_m, electrode_force_exponent, electrode_resistance_scale, faying_force_exponent, faying_resistance_scale, screening_peak_current_a`
- Mendeley force 기록값은 현재 1.5–6.0 kN contact-law 유효범위와 겹치지 않는다.

외부 test outcome은 parameter 선택이나 DOE에 사용하지 않았다.
