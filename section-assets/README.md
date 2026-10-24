# section-assets — 단면도(Section View) 사이트용 가구 SVG 라이브러리

기존 HOUSE 탭(`public/assets/house-objects/`, `index.html`)과 **완전히 분리된** 독립 라이브러리입니다.

```
section-assets/
  assets/svg/<파일명>/<뷰>.svg   뷰별 원본 선 SVG (front, rightSide, leftSide, top, back, section, unknown-N, loose-N)
  catalog/assets.json            메타데이터만 (SVG 내용은 없음)
  catalog/category-config.json   대분류/소분류
  logs/extract-raw.json          DWG별 추출 원시 로그(블록, 제외 요소, 플래그)
  logs/build-stats.json, report.md  집계와 보고서
  preview/index.html             검수용 미리보기(이미지는 loading=lazy, 클릭 시 상세)
  tools/                         extract.mjs(DWG→SVG), rules.py(분류 규칙), build_library.py, make_preview.py, make_report.py
```
재실행: `cd tools && npm install && node extract.mjs <DWG폴더> ../assets/svg && cp ../assets/extract-raw.json ../logs/ && python3 build_library.py && python3 make_fills.py && python3 make_preview.py && python3 make_report.py`
(extract.mjs 는 `<outDir>/../extract-raw.json` 을 씁니다.)

에셋 항목: assetId, nameKo/nameEn, categoryId/subcategoryId, tags, views.{front,leftSide,rightSide}(svg, viewBox, aspect, anchor), extraViews, defaultSizeCm, placement, classification{basis,confidence}, status, reviewReasons, errors.
- 정면/측면 SVG의 하단 중앙이 바닥 접지점(배치 기준점), 평면(top)은 중심.
- 로딩: `catalog/assets.json` 한 번 읽고, 화면에 보이는 에셋의 SVG 경로만 그때그때 로드하세요(전체 선로딩 금지).
- 원본 DWG 는 `/mnt/user-data/work/dwg-extract/` 에 보존(수정 없음).

- `*.fill.svg`: 정면/측면 선 SVG의 외곽 실루엣(흰 채움용). `tools/make_fills.py` 가 만들고 catalog 의 각 뷰에 `fill` 경로로 기록합니다(선 SVG는 수정하지 않음).

## 3차 수정 (2026-10-24)
- 좌우 반전: 한쪽 side 만 있던 에셋 전부에 반대쪽 파일 생성(`views.<side>.derived.mirroredFrom`). 선은 변형 없이 좌우만 뒤집음.
- 뭉친 시트 분리: 36개 시트 → 개별 에셋 `<원래ID>-1,-2,-3…`(왼쪽→오른쪽, 위→아래 순). 원본 시트 항목은 카탈로그/SVG 에서 제거(원본 DWG 는 그대로).
- 이름 변경/뷰 재지정/삭제/고양이 회전/선 정리는 `tools/r3_apply.py` 상단 설정표 참고. 적용 내역: `logs/r3-changes.md`.
- 추출기 버그 수정: DWG 의 '닫힌 폴리선'(libredwg flag 512) 마지막 변 누락, 같은 종류 뷰 파일 덮어쓰기, mini-fridge 의 바닥선이 지면선으로 오인되어 제거되던 문제.
- 전체 재생성: `node tools/extract.mjs <DWG폴더> <출력폴더>` 후 `sh tools/rebuild_all.sh <출력폴더> <출력폴더>/../extract-raw.json`


## 4차 수정 — 실루엣 다듬기
`tools/make_fills.py` 를 고쳐 `*.fill.svg` 가 (1) 부드럽게 다듬은 흰 채움 경로, (2) 바깥 윤곽만 담은 `<path class="o">` 를 함께 담습니다. 원본 선 SVG(`front.svg` 등)와 DWG 는 건드리지 않았습니다.
