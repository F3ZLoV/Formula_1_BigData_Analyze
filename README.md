# 🏎️ F1 Big Data Analytics & Circuit Characterization

## 📝 프로젝트 개요 (Project Overview)
이 프로젝트는 **2018년부터 2025년까지의 Formula 1 텔레메트리 데이터**를 대량으로 수집하여 MongoDB에 적재하고, 이를 바탕으로 각 **그랑프리 서킷의 주행 성향(Power Track vs. Technical Track)을 정량적으로 분석**하는 데이터 사이언스 프로젝트입니다.

단순한 기록 비교가 아닌, 랩 타임(Lap Time)에 영향을 미치는 물리적 변수(최고 속도, 코너링 속도)를 추출하여 **다중 선형 회귀(Multiple Linear Regression)** 모델을 통해 서킷별 중요도를 산출했습니다.

---

## 📂 파일 구성 (File Structure)

### 1. 📥 데이터 수집 (Data Collection)
* **파일명**: `Data_Collection.ipynb`
* **기능**: `FastF1` 라이브러리를 활용한 ETL 파이프라인 구축.
* **주요 특징**:
    * **Target Years**: 2018 ~ 2025 시즌 전체 데이터.
    * **DB 적재**: 수집된 데이터를 MongoDB(`f1_analytics_db`)에 구조화하여 저장.
        * `races_meta`: 경기 날짜, 서킷 정보, 날씨(기온, 노면 온도, 강우 여부).
        * `results`: 드라이버 순위, 포인트, 리타이어 여부.
        * `laps`: 랩 타임, 타이어 컴파운드, 섹터 기록.
        * `telemetry`: 초당 수집되는 차량의 속도, RPM, 기어, 스로틀/브레이크 입력값 (가장 방대한 데이터).
    * **스마트 복구(Smart Recovery)**: 수집 중단 시, 이미 DB에 존재하는 데이터는 건너뛰고 누락된 세션만 선별적으로 수집.
    * **장애 대응**: API 호출 실패 시 자동 재시도 및 로그 파일(`collection_failures.txt`) 기록.

### 2. 📊 서킷 특성 분석 (Circuit Analysis)
* **파일명**: `1_Analysis_Circuit_Importance.ipynb`
* **기능**: 서킷별 랩 타임 결정 요인 분석 (최고 속도 vs. 코너링 속도).
* **분석 방법론**:
    1.  **Feature Engineering**:
        * **Top Speed**: 퀄리파잉 폴 포지션(1위) 랩의 순간 최고 속도.
        * **Corner Speed**: 직선 주행을 배제하기 위해 텔레메트리 속도 분포 중 **하위 15%의 평균값**을 사용.
    2.  **Modeling**:
        * 서킷별로 `PoleTime ~ TopSpeed + CornerSpeed` 형태의 다중 선형 회귀 분석 수행.
        * 표준화(Standard Scaling)를 통해 변수 간 단위 차이를 제거하고 회귀 계수의 절대값 비율로 중요도(%) 산출.
    3.  **Visualization**: `Plotly`를 사용하여 파워 트랙과 테크니컬 트랙을 구분하는 Bar Chart 시각화.

---

## 🛠️ 기술 스택 (Tech Stack)

* **Language**: Python 3.x
* **Data Source**: [FastF1 API](https://github.com/theOehrly/Fast-F1)
* **Database**: MongoDB (Localhost:27017)
* **Libraries**:
    * Data Handling: `pandas`, `numpy`, `pymongo`
    * Machine Learning: `scikit-learn` (LinearRegression, StandardScaler)
    * Visualization: `plotly`

---

## 💡 주요 분석 결과 (Key Insights)

분석 결과, F1 서킷은 크게 세 가지 유형으로 분류되었습니다.

1.  **파워 트랙 (Power Tracks)**
    * **특징**: 공기 저항(Drag)을 줄이고 직선 가속력이 랩 타임에 절대적인 영향을 미침.
    * **해당 서킷**: 마이애미(Miami), 루사일(Qatar), 레드불링(Austria), 실버스톤(Silverstone).
    * *Note: 실버스톤은 고속 코너가 많으나, 감속 없이 풀 스로틀로 통과하는 구간이 많아 데이터상으로는 '속도'의 중요도가 높게 측정됨.*

2.  **밸런스 트랙 (Balanced Tracks)**
    * **특징**: 최고 속도와 코너링 성능의 중요도가 5:5 혹은 4:6으로 균형 잡힘. 차량의 종합적인 셋업(Setup) 능력이 중요.
    * **해당 서킷**: 바레인(Bahrain), 이몰라(Imola), 스파(Spa), 바르셀로나(Spain).

3.  **테크니컬 트랙 (Technical Tracks)**
    * **특징**: 최고 속도가 빠르더라도 코너링 성능이 떨어지면 좋은 기록을 낼 수 없음. 다운포스(Downforce) 중요도가 매우 높음.
    * **해당 서킷**: 헝가로링(Hungary), 스즈카(Japan), 모나코(Monaco).

### ⚠️ 흥미로운 발견 (Paradox)
* **몬자(Monza) & 바쿠(Baku)**:
    * 이들은 F1에서 가장 빠른 서킷들이지만, 데이터 분석 결과 **"코너링 중요도가 70% 이상"**으로 나타났습니다.
    * **원인**: 이 서킷에서는 모든 팀이 극단적인 Low Downforce(얇은 윙) 세팅을 하므로 직선 속도의 변별력이 거의 없습니다. 따라서 승부는 시케인(Chicane)과 같은 저속 코너를 누가 더 과감하게 공략하느냐(코너링)에서 갈리게 됩니다.

---

## 🚀 설치 및 실행 (Installation & Usage)

1.  **필수 라이브러리 설치**
    ```bash
    pip install fastf1 pandas numpy pymongo scikit-learn plotly
    ```

2.  **MongoDB 설정**
    * 로컬 환경에 MongoDB가 설치되어 있어야 하며, `mongodb://localhost:27017/` 포트에서 실행 중이어야 합니다.

3.  **데이터 수집 실행**
    * `Data_Collection.ipynb`를 실행하여 2018~2025년 데이터를 DB에 적재합니다. (데이터 양이 많아 시간이 소요될 수 있습니다.)

4.  **분석 실행**
    * 수집 완료 후 `1_Analysis_Circuit_Importance.ipynb`를 실행하여 분석 결과 및 시각화 차트를 확인합니다.

---

## 👤 Author
* Analyzed by **f3zlov**
* Based on F1 Big Data Analytics Project
