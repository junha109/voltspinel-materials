# XRD 시뮬레이터 · 분석 코드

볼트스피넬 머티리얼즈 · 소재개발팀 김준하 · 2026-11-19 (가상 기업 자료)

3분기 QC XRD 자료(`qc_xrd_2026Q3_406batches.zip`)를 만들고 현행 판정을 재현하는 코드다.

| 파일 | 하는 일 |
|---|---|
| `lnmo_xrd.py` | 3상 구조(무질서 Fd-3m · 규칙 P4₃32 · rock-salt LixNi1-xO Fm-3m) + Cu Kα1 · Kα2 분말 XRD 시뮬레이터. 구조인자(원자 산란 인자 · 이상 분산), 결정 크기 · 미세 변형 피크 폭, Mn 형광 배경, 시료 높이 오차 |
| `generate_dataset.py` | 가상 공정 모델(1차 소성 온도 · 후어닐링 · 냉각 · 소성로 · 조성 → rock-salt 함량 · 규칙화 · 격자) + 406배치 2분 · 10분 패턴 생성 |
| `baseline_searchmatch.py` | 자료 생성 때 함께 쓴 간이 search-match (백분위 배경, 평활 없음) |
| `current_qc_searchmatch.py` | 현행 QC 판정 재현 (SOP-QC-XRD-03: 평활 · SNIP 배경 · 4σ · 최댓값 2 % · DB NiO ±0.2°). LIMS의 「현행 QC 판정」이 이 코드의 결과다 |

## 실행

```bash
pip install numpy pandas scipy
python generate_dataset.py --out ../data          # 정답 seed 20261105, 측정 잡음 seed 20261106 · 수 분
python current_qc_searchmatch.py --data ../data   # 현행 판정 재현 → current_qc_2min.csv
```

## 자료 구성 (`../data`)

- `patterns_2min/VS26Q3-NNN.xy` — 일상 QC 2분 스캔, 2θ 10–90° · 0.02°
- `patterns_10min/VS26Q3-NNN.xy` — 같은 시료의 10분 정밀 스캔 (계수 약 5배)
- `labels.csv` — 레시피 · ICP 조성 · 정밀 재분석(Rietveld) · 외부 중성자 회절 규칙상 분율 · 측정 메타
- VS26Q3-401–406은 분석평가팀이 제품 유형별로 고른 대표 배치다.

## 공정 모델 (사내 경험식 기반)

- rock-salt ≈ [0.2 + 6.0 / (1 + exp(−(T − 900)/30))] × exp(−t후어닐링/18) × 조성 보정(Ni 과잉 · Li 부족이면 증가)
- 규칙상 분율 ≈ f0 + (1 − f0)(1 − exp(−t후어닐링/20)), f0 = 0.25(서냉) / 0.05(급냉), 1차 소성 ≥ 950 °C면 f0 × 0.6
- 소성로별 실제 온도 차이: 1호기 0 · 2호기 +3.5 · 3호기 −2.5 °C

## 측정 조건

- XRD-1은 기본 검출기 설정이라 Mn 형광 배경이 스피넬 (400) 높이의 0.6–1.2배, XRD-2는 형광 억제 설정으로 0.12–0.25배다.
- 시료 높이 오차는 표준편차 0.08 mm(측각기 반경 150 mm)다.
- 측정 조건은 공개 실측 자료(A-Lab Precursor Genome, CC BY 4.0)의 통계에 맞췄다.

> 수업 프로젝트용 가상 기업 자료다. 결정 구조는 공개 결정구조와 문헌 경향으로, 정답 값은 가상 공정 모델로 만들었다. 실측 데이터나 특정 기관의 데이터를 정답으로 쓰지 않았다.
