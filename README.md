# F1 Miami GP Pit Stop Analysis
운영 데이터 기반 의사결정 모델링 | Regression Modeling of Pit Stop Decisions from Race Telemetry

![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/Miami_Circuit.png)

## 비즈니스 맥락 | Business Context

F1 피트스톱은 수십 초 안에 이루어지는 고압적 운영 의사결정입니다. 타이밍이 레이스 결과를 바꿉니다.
이 프로젝트의 핵심 질문은 데이터 분석가가 어떤 도메인에서든 마주치는 질문과 같습니다:

"관찰 가능한 성능 저하 신호로 의사결정 시점을 예측할 수 있는가?"

레이스 텔레메트리 데이터를 활용해 탐색적 분석 → 가설 수립 → 회귀 모델링 → 교차검증 기반 모델 선택의 전체 분석 파이프라인을 구현했습니다. 비정형적이고 고차원적인 데이터에서 의미 있는 패턴을 추출하고, 이를 해석 가능한 모델로 연결하는 역량을 보여주는 프로젝트입니다.

In Formula 1, pit stop timing is a high-stakes decision where a few seconds determine race outcomes.
The core analytical question mirrors what data analysts face across domains:

"Can observable performance degradation signals predict when a decision should be made?"

Using race telemetry data, this project implements a full analysis pipeline — EDA → hypothesis formation → regression modeling → cross-validation-based model selection — demonstrating the ability to extract meaningful patterns from complex, high-dimensional data and translate them into interpretable, actionable models.


### Project Objective
In Formula 1, pit stops are high-pressure operational events that can significantly influence race outcomes.

While average pit stop time is often reported, operational performance is multidimensional — shaped by speed, consistency, variability, and timing within race strategy.

This project aims to answer:

**How do pit stop time distributions and variability differ across teams, and what does that reveal about operational efficiency?**

Rather than focusing only on mean performance, this analysis investigates structural variation in pit stop execution.

## 주요 결과 | Key Findings
- 랩타임 +1초 → 피트스톱 확률 약 14.7% 증가 (로지스틱 회귀 오즈비 기준)

  +1 second in lap time → pit stop odds increase by ~14.7% (logistic regression odds ratio)
- 타이어 수명 +1랩 → 피트스톱 확률 약 31.7% 증가 — 타이어 마모가 가장 강한 예측 변수

  +1 lap of tire age → pit stop odds increase by ~31.7% — tire degradation is the strongest predictor
- 타이어 컴파운드 효과: SOFT 타이어는 HARD 대비 피트스톱 오즈 6.43배 높음

  Compound effect: SOFT tires show 6.43× higher odds of pitting vs HARD
- 모델 선택: 선형 확률 모델(Model 1)이 k-fold 교차검증(k=5,10,20) 전반에서 가장 낮은 MAE 달성 → 단순한 모델이 더 잘 일반화됨

  Model selection: Linear Probability Model (Model 1) achieved lowest MAE across all k-fold CV configurations — simpler model generalized better
- 피트스톱은 랩 번호나 컴파운드보다 성능 저하 신호(랩타임·타이어 마모)에 의해 주도됨을 확인

  Pit decisions are primarily driven by performance degradation signals, not race stage or compound alone

## 모델 비교 요약 | Model Comparison
 
### 선형 확률 모델 교차검증 결과 | Linear Model Cross-Validation
 
| 모델 / Model | In-Sample MAE | 5-fold CV | 10-fold CV | 20-fold CV | 선택 |
|---|---|---|---|---|---|
| **Model 1** (lap_time, tyre_life) | **0.05045** | **0.05073** | **0.05100** | **0.05086** | ✅ 최종 선택 |
| Model 2 (+ lap_number, compound) | 0.05975 | 0.05922 | 0.05939 | 0.05925 | — |
 
> **선택 이유:** Model 2는 변수를 추가했음에도 일반화 성능이 개선되지 않음. 피트스톱 결정은 추가적인 맥락 변수보다 성능 저하 지표로 주로 설명됨.
>
> Model 2 added complexity but did not improve generalization. Pit decisions are primarily driven by measurable performance degradation.
 
### 로지스틱 vs 선형 확률 모델 비교 | Logistic vs Linear
 
| 항목 / Aspect | 로지스틱 회귀 / Logistic | 선형 확률 모델 / Linear |
|---|---|---|
| Probability bounds | Always between 0 and 1 | Can exceed bounds        |
| Interpretation     | Odds ratios            | Direct marginal effects  |
| Suitability        | Binary outcomes        | Approximation            |

## Datasets
Source: Miami GP 2024 Race Logs (1,111 laps, 32 indicators) from `f1dataR` R package
- 1,111랩 × 32개 변수 레이스 텔레메트리 데이터
- https://docs.fastf1.dev/
- https://cran.r-project.org/web/packages/f1dataR/f1dataR.pdf
- Data compiled from official Formula 1 race timing sheets and lap records.

The dataset includes:
- Lap numbers
- Pit stop times (seconds)
- Driver details
- Tire information
- Track status

  ![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/lap_dat.png)

This project aims to evaluate the structure of performance variance rather than to predict any single outcome.

Variables used:
- `lap_time`: recorded time to complete a lap (seconds)
- `lap_number`: lap number from which the telemetry data was recorded (number of laps)
- `tyre_life`: number of laps completed on a set of tires (number of laps)
- `compound`: type of tire used (SOFT, MEDIUM, HARD)
- `pit_in`: whether a driver made a pit stop during a lap (binary: 0 = no pit stop, 1 = pit stop occured)

## Analytical Framework

### Step 1. Data Processing
Data processing steps included:
- Selecting relevant variables
- Removing missing lap time values (<0.1% of total)
  ![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/missing_data.png)
  - Missing lap times were primarily associated with undefined track status codes or race incidents.

### Step 2. 탐색적 데이터 분석 | Exploratory Data Analysis (EDA)
Before comparative analysis, I conducted structural exploration:

1. **랩별 피트스톱 빈도 | Pit Stop Frequency by Lap**
   
    A bar plot was used to visualize the number of pit stops by lap number.
  ![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/barplot.png)
  - Key observation:
    - Most pit stops occurred during the first half of the race.
    - A clear peak appeared around Lap 28.
    - Later-race pit stops likely reflect two-stop strategies or fastest-lap attempts.
  - This confirms that pit stop timing is not random but strategically clustered.
  - 피트스톱은 랩 28 전후에 집중 — 전략적으로 클러스터링된 패턴 확인 (무작위 분포 아님)

2. **팀별 랩타임 분포 | Lap Time Distribution by Team**
    
    Density plots were generated to compare lap time distributions across teams.
  ![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/densityplot.png)
  - Key observation:
    - Most lap times were under 100 seconds.
    - Certain teams (e.g., Mercedes, Williams) displayed occasional high-end outliers.
    - Lower lap times indicate stronger on-track performance.
  - Variation in distribution width suggests performance consistency differences between teams.
  - 분포 폭의 차이가 팀 간 운영 일관성 차이를 반영

3. **타이어 컴파운드별 수명 | Tire Life by Compound**

    A boxplot was used to examine tire longevity across compounds.
  ![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/boxplot.png)
  - Key observation:
    - Hard tires lasted slightly longer than medium tires.
    - Soft compound data was limited in this race.
    - Results reflect the performance–durability trade-off inherent in tire strategy.
    - HARD > MEDIUM 순으로 타이어 수명이 길며, 내구성-성능 트레이드오프 확인

4. **Tire compound usage**

    A frequency bar chart was constructed to assess compound usage.
  ![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/barplot_cpd.png)
  - Key observation:
    - Medium tires were most frequently used.
    - Strategic choice reflects balance between durability and competitive lap time.
   
#### EDA Summary
The exploratory phase revealed three important structural insights:
1. Pit stops cluster around strategic race windows rather than being evenly distributed.
2. Lap time variability differs across teams, suggesting operational performance gaps.
3. Tire compound selection reflects a performance–durability trade-off influencing pit timing.

These findings motivated the subsequent regression modeling to formally test whether lap time, tire life, compound, and race progression predict pit stop decisions.

## 모델링 | Modeling Approach
1. 선형 확률 모델 (LPM) — 방향성 초기 검증, 교차검증 기반 모델 선택
2. 로지스틱 회귀 — 오즈비 해석, 시나리오별 피트스톱 확률 예측
3. k-fold 교차검증 (k = 5, 10, 20) — 과적합 방지 및 일반화 성능 평가

## Linear Regression Analysis

Following EDA, predictive modeling was conducted to evaluate whether pit stop decisions could be statistically explained by race dynamics and tire conditions.

The response variable:

`pit_in` (binary)
- 0 = No pit stop
- 1 = Pit stop occurred

### Model 1: Linear Probability Model

The first model examined whether pit stops were associated with lap time and tire age:

$$
\mathbb{E}(\text{pit in} \mid \text{lap time}, \text{tyre life})
$$

Predictors: `lap_time`, `tyre_life`

Purpose:
To test whether slower laps and aging tires increase pit stop probability.

![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/first_linear_model.png)

**Interpretation**:
- Positive coefficient for `lap_time` → slower laps increase likelihood of pit stop.
- Positive coefficient for `tyre_life` → older tires increase likelihood of pit stop.

This model provided initial directional evidence linking performance degradation to pit decisions.

### Model 2: Extended Linear Model

The second model incorporated additional race context:

$$
\mathbb{E}(\text{pit in} \mid \text{lap time}, \text{lap number}, \text{compound}, \text{tyre life})
$$

Additional predictors: `lap_number`, `compound` (categorical)

Purpose:
To evaluate whether race progression and tire compound influence pit strategy.

![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/second_linear_model.png)

**Interpretation**:
- `lap_time` and `tire_life` remained strong predictorst.
- Negative coefficient for `lap_number` → more laps decrease likelihood of pit stop.
- `compound` had a small and non-significant effect → tire compound did not meaningfully influence pit stop decisions when other factors were considered.

This model allowed assessment of:
- Whether drivers pit more frequently at certain race phases.
- Whether tire compound materially shifts pit probability.

### Cross-Validation Framework
To evaluate model generalization and avoid overfitting, I implemented k-fold cross-validation under multiple configurations (k = 5, 10, 20).

**Objective**: Estimate out-of-sample prediction error and compare model stability.

Evaluation metric:
- **Mean Absolute Error (MAE)**: An estimator measures the absolute difference between the predicted values and the actual values in the dataset.

| Model   | In-Sample MAE | 5-fold CV | 10-fold CV | 20-fold CV |
| ------- | ------------- | --------- | ---------- | ---------- |
| Model 1 | 0.05045       | 0.05073   | 0.05100    | 0.05086    |
| Model 2 | 0.05975       | 0.05922   | 0.05939    | 0.05925    |

#### Key Observations
1. Model 1 consistently achieved lower MAE across all k values.
2. Increasing k did not materially change performance rankings.
3. Model 2 added complexity but did not improve generalization.
4. Standard deviations across folds remained low, indicating stable performance.

#### Final Selected Model
Model 1 was selected based on:
- Lower cross-validated MAE
- Simpler structure
- Better generalization performance

Final specification:

$$ \mathbb{E}(\text{pit in} \mid \text{lap time}, \text{tyre life})$$

This suggests that pit decisions are primarily driven by measurable performance degradation rather than race progression variables.


## Logistic Regression Analysis

Response variable:
- `pit_in` (0 = no stop, 1 = stop)

Predictors:
- `lap_time`
- `lap_number`
- `tyre_life`
- `compound`

$$
\log(odds(\text{pit in} \mid \text{lap time}, \text{lap number}, \text{compound}, \text{tyre life}))
$$

![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/log_regression.png)

**Interpretation of Odds Ratios**:
- `lap_time`: +1 second increase → odds of pitting increase by ~14.7%
- `lap_number`: +1 additional lap → odds decrease by ~14.8%
- `tyre_life`: +1 additional lap on current tires → odds increase by ~31.7%
- `compound`:
  - MEDIUM vs HARD → 1.64× higher odds
  - SOFT vs HARD → 6.43× higher odds

These results confirm that tire degradation and compound selection significantly influence pit probability.

### 시나리오별 예측 | Scenario-Based Prediction

| 시나리오 | 조건 | 예측 피트스톱 확률 |
|---|---|---|
| 극단적 저하 | 높은 랩타임 + 레이스 후반 + SOFT 타이어 노후화 | **~73%** |
| 중간 조건 | 레이스 중반 + HARD 타이어 | **~50%** |

Under extreme race conditions:
- High lap time
- Late race stage
- Aged SOFT tires

![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/log_pred1.png)

Predicted pit probability ≈ 73%

Under mid-race HARD tire conditions:

![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/log_pred2.png)

Predicted probability ≈ 50%

This highlights:
- Logistic regression captures directional influence
- However, prediction confidence remains moderate due to strategic uncertainty inherent in motorsport

## Logistic vs Linear Regression
| Aspect             | Logistic Regression    | Linear Probability Model |
| ------------------ | ---------------------- | ------------------------ |
| Probability bounds | Always between 0 and 1 | Can exceed bounds        |
| Interpretation     | Odds ratios            | Direct marginal effects  |
| Suitability        | Binary outcomes        | Approximation            |

Logistic regression is theoretically more appropriate for binary decisions, while linear models provided simpler comparative benchmarks during model selection.

## Conclusion

This project demonstrates a full modeling pipeline:
- Structured EDA
- Model specification
- Cross-validation across multiple k values
- Comparative model selection
- Probability modeling via logistic regression

**Key insight**:

Pit stop timing is statistically associated with performance degradation variables (lap time and tire age), while additional contextual variables add limited predictive value.

However, strategic race decisions introduce inherent uncertainty, limiting predictive precision — a realistic reflection of high-stakes decision environments.

## 한계 및 고려사항 | Limitations
 
- 단일 레이스(2024 Miami GP) 데이터 — 다른 서킷·시즌으로의 일반화 제한

  Single-race dataset (2024 Miami GP) — generalizability to other circuits and seasons is limited
- 피트스톱에는 세이프티카, 팀 전략 지시 등 모델에 미포함된 외생 변수 존재

  Unmodeled exogenous variables exist (e.g., safety car deployments, team radio strategy calls)
- 예측 정밀도는 모터스포츠의 전략적 불확실성으로 인해 본질적 한계 있음

  Predictive precision has inherent limits due to the strategic uncertainty in motorsport decision-making
- 향후 개선: 멀티 레이스 데이터 통합, 팀별 전략 패턴 고정효과 추가

  Future improvements: multi-race data integration, team-level fixed effects for strategy patterns
