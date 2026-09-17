# 점용접 데이터 디렉터리

원본 외부 데이터와 생성된 시공간 field는 Git에 commit하지 않는다.

권장 로컬 구조는 다음과 같다.

```text
data/
├── raw_external/          # 원본 archive, read-only
├── raw_simulation/        # solver 출력
├── processed_simulation/  # ML tensor
├── processed_external/    # 안전한 변환 형식
└── manifests/             # 출처·hash·split·변환 기록
```

`raw_external`과 `processed_external`은 simulation manifest를 재사용하지 않는다. 외부
데이터를 받기 전 license와 checksum을 먼저 기록한다.

## 외부 데이터 다운로드

기본 명령은 Mendeley의 `Data_RSW.csv`와 commit으로 고정한 Polito CSV를 내려받고 각
파일의 크기와 checksum을 검증한다.

```bash
python scripts/download_rsw_external_data.py --source all
```

Mendeley IR/RGB 이미지 1,485장은 초기 tabular·시계열 분석에는 필요하지 않다. 이미지
실험을 시작할 때만 전체 archive를 추가한다.

```bash
python scripts/download_rsw_external_data.py --source mendeley --include-images
```

원본은 `data/raw_external/`, provenance와 계산된 SHA-256은
`data/manifests/external_download_manifest.json`에 저장된다. 원본은 `.gitignore` 대상이다.
