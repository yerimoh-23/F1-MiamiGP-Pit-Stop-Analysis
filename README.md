# F1 Miami GP Pit Stop Analysis
Multivariate Analysis of Formula 1 Pit Stop Performance (R & Data Visualization)

![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/Miami_Circuit.png)

## Project Objective
In Formula 1, pit stops are high-pressure operational events that can significantly influence race outcomes.

While average pit stop time is often reported, operational performance is multidimensional — shaped by speed, consistency, variability, and timing within race strategy.

This project aims to answer:

**How do pit stop time distributions and variability differ across teams, and what does that reveal about operational efficiency?**

Rather than focusing only on mean performance, this analysis investigates structural variation in pit stop execution.

## Datasets
Source: Miami GP 2024 Race Logs (1,111 laps, 32 indicators) from `f1dataR` R package
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

### Step 2. Exploratory Data Analysis (EDA)
Before comparative analysis, I conducted structural exploration:

1. **Pit stop frequency across laps**
   
    A bar plot was used to visualize the number of pit stops by lap number.
  ![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/barplot.png)
  - Key observation:
    - Most pit stops occurred during the first half of the race.
    - A clear peak appeared around Lap 28.
    - Later-race pit stops likely reflect two-stop strategies or fastest-lap attempts.
  - This confirms that pit stop timing is not random but strategically clustered.

2. **Lap time distribution by team**
    
    Density plots were generated to compare lap time distributions across teams.
  ![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/densityplot.png)
  - Key observation:
    - Most lap times were under 100 seconds.
    - Certain teams (e.g., Mercedes, Williams) displayed occasional high-end outliers.
    - Lower lap times indicate stronger on-track performance.
  - Variation in distribution width suggests performance consistency differences between teams.

3. **Tire life by compound**

    A boxplot was used to examine tire longevity across compounds.
  ![alt text](https://github.com/yerimoh-23/F1-MiamiGP-Pit-Stop-Analysis/blob/main/website_img/boxplot.png)
  - Key observation:
    - Hard tires lasted slightly longer than medium tires.
    - Soft compound data was limited in this race.
    - Results reflect the performance–durability trade-off inherent in tire strategy.

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

### Scenario-Based Prediction
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


