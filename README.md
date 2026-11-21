[README.md](https://github.com/user-attachments/files/35205378/README.md)
# AI-M1-1 🌡️ 서울 기온의 장기 변화 분석 (2000–2024)

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![pandas](https://img.shields.io/badge/pandas-2.x-150458?logo=pandas&logoColor=white)
![Streamlit](https://img.shields.io/badge/Dashboard-Streamlit-FF4B4B?logo=streamlit&logoColor=white)
![Data](https://img.shields.io/badge/Data-ERA5%20%7C%20CC%20BY%204.0-2F6DB5)

> 서울의 일별 기온 25년치(9,000여 일)를 **추세 · 계절성 · 노이즈**로 분해해, "서울은 정말 더워졌는가?"를 데이터로 검증하는 시계열 분석 프로젝트입니다.

📄 **분석 보고서 전문: [REPORT.md](REPORT.md)**

---

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [분석 질문](#2-분석-질문)
3. [주요 결과 미리보기](#3-주요-결과-미리보기)
4. [데이터](#4-데이터)
5. [분석 방법](#5-분석-방법)
6. [빠른 시작](#6-빠른-시작)
7. [대시보드](#7-대시보드-보너스)
8. [프로젝트 구조](#8-프로젝트-구조)
9. [한계](#9-한계)
10. [AI 활용 및 라이선스](#10-ai-활용-및-라이선스)

---

## 1. 프로젝트 개요

| 항목 | 내용 |
|---|---|
| **목적** | 서울 기온의 장기 변화를 정량화하고, 변화가 계절과 극단 기온에 어떻게 나타나는지 파악 |
| **데이터** | Open-Meteo Historical Weather API (ECMWF ERA5 재분석), 2000-01-01 ~ 2024-12-31, 일 단위 |
| **규모** | 9,132일 × 5개 변수 (최고·최저·평균 기온, 강수량) |
| **핵심 기법** | 이동평균, 연 단위 선형 추세, 구간별 통계, 변화량·변동성, 임계값 초과 일수, 시계열 분해, 베이스라인 예측 |
| **산출물** | 분석 보고서 1부, 그림 7개, 요약 통계, 대화형 대시보드 |

## 2. 분석 질문

| ID | 질문 | 접근 |
|---|---|---|
| Q1 | 지난 25년간 서울 기온은 상승했는가? 얼마나 빠른가? | 365일 이동평균, 연평균 선형 회귀 |
| Q2 | 기온 변화는 계절(월)마다 같은가? | 2000–09년 vs 2015–24년 월평균 비교 |
| Q3 | 폭염일·열대야 같은 극단 기온일은 늘었는가? | 기상청 특보 기준 임계값 초과 일수 |
| Q4 | 하루 단위 기온 변동은 언제 가장 큰가? | 전일 대비 변화량의 월별 표준편차 |

## 3. 주요 결과 미리보기

결과 수치와 인사이트는 [REPORT.md 요약](REPORT.md#요약-executive-summary)에서 볼 수 있습니다.

| 추세·계절성·노이즈 | 연평균 기온 추세 |
|---|---|
| ![daily](images/01_daily_temperature_moving_average.png) | ![annual](images/02_annual_mean_trend.png) |
| **월별 변화 비교** | **극단 기온일** |
| ![monthly](images/03_monthly_period_comparison.png) | ![extremes](images/04_extreme_days_per_year.png) |

<details>
<summary>그림 더 보기 (변동성 · 시계열 분해 · 베이스라인 예측)</summary>

![volatility](images/05_daily_change_volatility.png)
![decomposition](images/06_decomposition.png)
![forecast](images/07_baseline_forecast.png)

</details>

## 4. 데이터

| 항목 | 내용 |
|---|---|
| 출처 | [Open-Meteo Historical Weather API](https://open-meteo.com/en/docs/historical-weather-api) |
| 원천 | ECMWF ERA5 재분석 자료 (약 25km 격자) |
| 지점 | 서울시청 부근 (37.5665°N, 126.9780°E) |
| 기간 | 2000-01-01 ~ 2024-12-31 (Asia/Seoul) |
| 수집 | `fetch_data.py` — API 키 불필요, 표준 라이브러리만 사용 |
| 상세 | [data/README.md](data/README.md) — 변수 정의, 요청 URL, 라이선스 |

**정제 기준 요약**

- 중복 날짜를 제거하고, 날짜를 재정렬해 시간 축을 연속으로 만들었습니다.
- 물리적으로 불가능한 값(-35℃ 미만, 45℃ 초과)과 최저기온이 최고기온보다 높은 논리 오류는 결측으로 처리했습니다.
- **연속 3일 이하** 공백만 선형 보간하고, 더 긴 공백은 그대로 두었습니다.
- 평년 대비 ±4σ를 벗어난 값은 **표시만 하고 유지**했습니다. 극한 기상 자체가 분석 대상이기 때문입니다.

## 5. 분석 방법

```
원자료 ─▶ 정제 ─▶ 특성 생성 ─┬─▶ 이동평균 (30일·365일)        ─▶ Q1
                              ├─▶ 연 단위 집계 + 선형 회귀       ─▶ Q1
                              ├─▶ 구간별 월평균 비교             ─▶ Q2
                              ├─▶ 임계값 초과 일수               ─▶ Q3
                              ├─▶ 전일 대비 변화량 · 변동성      ─▶ Q4
                              ├─▶ 시계열 분해 (추세+계절+잔차)   ─▶ 보너스
                              └─▶ 베이스라인 예측 4종 비교       ─▶ 보너스
```

| 설계 결정 | 근거 |
|---|---|
| %변화율 대신 **℃ 단위 변화량** 사용 | 기온은 0℃를 지나므로 비율 변화가 의미 없음 (0.5→1℃ = +100%) |
| ERA5 **단일 모델**로 전 기간 수집 | 기본값은 2017년 이후 고해상도 모델이 섞여 연도 간 비교가 왜곡됨 |
| 두 비교 구간 사이에 **5년 간격** | 경계 연도가 결과를 좌우하지 않도록 함 |
| 이동평균의 **앞부분 제외** | 창이 다 차지 않은 구간은 일부 계절만 반영해 편향됨 |

## 6. 빠른 시작

**요구 사항:** Python 3.10 이상, 인터넷 연결 (데이터 수집 시)

```bash
git clone https://github.com/<사용자명>/seoul-temperature-analysis.git
cd seoul-temperature-analysis

python -m pip install -r requirements.txt
python fetch_data.py       # ① 데이터 수집 → data/
python analysis.py         # ② 분석 → images/, results/, REPORT.md
```

| 단계 | 소요 시간 | 산출물 |
|---|---|---|
| `fetch_data.py` | 수 초 | `data/seoul_weather_2000_2024.csv`, `data/metadata.json` |
| `analysis.py` | 10~30초 | `images/*.png` (7개), `results/summary.json`, `results/annual_stats.csv`, `REPORT.md` |

<details>
<summary>Windows에서 <code>pip.exe</code>가 차단되거나 데이터 수집이 실패할 때</summary>

- `pip install` 대신 `python -m pip install --user -r requirements.txt`로 실행하세요.
- `python`을 인식하지 못하면 `py`로 바꿔 실행하세요.
- `fetch_data.py`가 실패하면 화면에 나온 URL을 브라우저에서 열고, 내용을 `data/open_meteo_raw.json`으로 저장한 뒤 다시 실행하세요.

</details>

## 7. 대시보드 (보너스)

```bash
python -m streamlit run dashboard.py
```

브라우저(`http://localhost:8501`)에서 아래 조건을 바꿔 가며 결과를 탐색할 수 있습니다.

| 조건 | 탐색 예시 |
|---|---|
| 기간 (연도 범위) | 2000–2009 → 2015–2024로 바꿔 폭염일·열대야 비교 |
| 지표 (평균·최고·최저) | 7·8월 + 최저기온으로 여름밤 기온 추세 확인 |
| 이동평균 창 (7~365일) | 7 → 30 → 365일로 늘려 노이즈 → 계절성 → 추세 순으로 확인 |
| 폭염·열대야 기준 | 31℃, 35℃로 바꿔 결론이 기준에 따라 유지되는지 확인 |
| 월 선택 | 특정 계절만 골라 연도별 추세 비교 |

## 8. 프로젝트 구조

```
seoul-temperature-analysis/
├── data/
│   ├── seoul_weather_2000_2024.csv   # 원자료 (fetch_data.py 생성)
│   ├── metadata.json                 # 요청 URL · 단위 · 라이선스
│   └── README.md                     # 데이터 명세
├── images/                           # 그림 7개 (analysis.py 생성)
├── results/
│   ├── summary.json                  # 전체 분석 수치
│   └── annual_stats.csv              # 연도별 통계
├── fetch_data.py                     # 데이터 수집
├── analysis.py                       # 정제 · 분석 · 시각화 · 보고서 생성
├── dashboard.py                      # Streamlit 대시보드
├── REPORT.md                         # 분석 보고서
├── requirements.txt
└── README.md
```
<img width="2323" height="2017" alt="시각화 증명사진 1" src="https://github.com/user-attachments/assets/c87c8bd5-7fa5-4f8e-a54e-583546dc590d" />
<img width="1872" height="1155" alt="시각화 증명사진 2" src="https://github.com/user-attachments/assets/b2179ae4-446e-4b7e-bcf6-54feea6f8418" />

## 9. 한계

- **격자 자료:** ERA5는 약 25km 격자 평균이라 도심 열섬과 극한값이 약하게 나타납니다. 극단 기온일의 절대값은 기상청 발표와 다를 수 있습니다.
- **기간:** 25년은 수십 년 주기의 자연 변동과 장기 추세를 분리하기에 짧습니다.
- **인과:** 이 분석은 변화의 크기와 형태를 보여 줄 뿐, 원인을 분리하지는 않습니다.

자세한 내용과 향후 과제는 [REPORT.md 8장](REPORT.md#8-한계-및-향후-과제)에 있습니다.

## 10. AI 활용 및 라이선스

**AI 활용:** 코드 작성, 분석 기법 검토, 해석 초안 작성에 생성형 AI(Claude)를 보조 도구로 사용했습니다. 작업별 사용 목적과 검증 방법은 [REPORT.md 10장](REPORT.md#10-ai-활용-내역)에 기록했습니다.

**데이터 라이선스**
- Weather data by [Open-Meteo.com](https://open-meteo.com/), licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- Generated using Copernicus Climate Change Service information (ERA5)
- Open-Meteo 무료 API는 비상업적 용도로만 사용할 수 있습니다.

**인용**
- Zippenfenig, P. (2023). *Open-Meteo.com Weather API*. Zenodo. https://doi.org/10.5281/ZENODO.7970649
- Hersbach, H. et al. (2020). The ERA5 global reanalysis. *Q. J. R. Meteorol. Soc.*, 146, 1999–2049. https://doi.org/10.1002/qj.3803
