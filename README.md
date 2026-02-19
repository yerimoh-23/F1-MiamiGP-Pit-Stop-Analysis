# F1 Miami GP Pit Stop Analysis
Multivariate Analysis of Formula 1 Pit Stop Performance (R & Data Visualization)

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
   

