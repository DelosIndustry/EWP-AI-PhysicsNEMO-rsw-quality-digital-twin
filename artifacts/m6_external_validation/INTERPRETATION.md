# M6 외부 데이터 adapter 해석

## 결론

- 원본 checksum, 변환 파일 checksum, 두 manifest ID와 split 누수 검사가 모두 통과했다.
- Mendeley 4186행은 독립 용접 495개로
  변환했다. 평가는 CSV 행이 아니라 `sample_id` 단위다.
- 정적 값 충돌은 5개 weld,
  7개 field-weld pair다. 충돌 scalar는 비워 두고
  min/max/unique count/flag를 함께 저장했으므로 임의의 첫 값을 정답으로 쓰지 않는다.
- Polito 1976행은 원본 순서의 `sample_row_id`를 보존했다.
  metadata key 중복이 있어도 key join을 하지 않는다.
- Polito 신호는 공개된 0~1 정규화 값이다. 전압·전류·가압력의 SI 단위로 역추정하지 않는다.

## 그림 읽는 법

1. `mendeley_split_ctq.png`: 왼쪽 막대에서 희귀 Bad/Explode가 development와 test에 모두
   남았는지 확인한다. 오른쪽은 nugget 직경과 pull force의 실측 관계이며 빈 label은 충돌
   때문에 의도적으로 제외된 값이다.
2. `mendeley_conflicts.png`: 어떤 필드와 Sample ID에서 서로 다른 값이 보고됐는지 확인한다.
3. `polito_group_split.png`: 세 split의 표본·fault 수와 Car Body 격리를 확인한다.
4. `polito_test_signal_examples.png`: test의 정상/불량 한 건을 비교한다. 선이 끝난 뒤의 0은
   측정값이 아니라 padding이므로 mask 밖이다.

## 사용 경계

Mendeley manifest의 종류는 `experimental_ctq`, Polito manifest의 종류는
`factory_timeseries`다. 둘 다 simulation의 내부 온도장 `T(r,z,t)` label이 아니다. 따라서
M5 FNO의 field NRMSE와 이 데이터의 nugget/fault metric을 하나의 평균 점수로 합치지 않는다.
