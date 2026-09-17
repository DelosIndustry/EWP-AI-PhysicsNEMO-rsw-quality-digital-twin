# M2.7 결과 해석

## 한 줄 결론

코드와 데이터 무결성 검사는 통과했지만, force 단위와 센서 전달경로가 확인되지 않아 물리 접촉법칙 보정과 RSW-SIM-V3 생성은 계속 차단된다. 대신 force를 제외한 Mendeley development CTQ 기준선은 진행할 수 있다.

## 왜 단위를 자동 변환하지 않았나

- 같은 signal의 공개 표기가 `n, psi`로 충돌한다.
- 알려진 질량 검사는 0–2128 g이며 RMSE는 `2.828 g`다.
- 최대 질량의 중력 환산값 `20.869 N`은 bench reference일 뿐, 용접 전극의 archived signal 환산 계수가 아니다.
- 압력에서 tip force를 얻으려면 piston 면적, 기계 전달비, 효율/마찰, 측정 위치가 필요하다. 전극 tip 면적을 piston 면적 대신 쓰면 안 된다.

## 시계열에서 확인한 것

- development: `396` weld / `3378` records
- 비양수·결측 current/force record: `1` / `14` (삭제·대체하지 않고 flag 보존)
- nominal 10 Hz inclusive 표본수와 불일치한 weld: `134`
- mean(I²)/mean(I)² 중앙값: `1.029602`

Jensen 부등식으로 mean(I²) ≥ mean(I)²이다. 따라서 weld당 평균 current 하나만 쓰면 within-weld 변동을 잃는다. 그러나 전압·저항·정확한 timestamp가 없으므로 이 비율을 실제 Joule energy 또는 에너지 오차라고 부르지는 않는다.

## 두 갈래의 다음 단계

1. Lab CTQ track: force를 기본 feature에서 제외하고 development weld만으로 nugget/pull/category 기준선을 만든다. test는 checkpoint 고정 후 한 번만 연다.
2. Automotive physical-twin track: 동기화된 SI V/I/F, 원 timestamp, stack/coating/electrode metadata를 확보하거나 제안된 coupon protocol을 수행하기 전까지 보정을 차단한다.

후보 데이터셋의 웹 설명은 raw audit와 다르다. IEEE 후보의 동기화 신호는 catalog claim이며 login 뒤 sample·단위·license를 직접 확인하기 전에는 사용 가능으로 승격하지 않았다.
