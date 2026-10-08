# Formative Assessment 7 README

**Group 24**

**Authors:** Pinili, Nicklaus Vincent · Tan, Heinz Nicole · Valdez, Chance Jayden

## Video Presentation

**Presentation Link:** PASTE YOUTUBE LINK HERE

## What is in this project?

| Part | Topic | Distribution | Output |
|---|---|---|---|
| Part 1 | Waiting time between posts in the One Piyu Community (OPC) Facebook group | Exponential | `FA7_Part1_Group24.pdf` (Report) |
| Part 2 | Occupancy proportion of selected facilities at Far Eastern University (FEU) | Normal | `FA7_Part2_Group24.pdf` (LaTeX Beamer Slides) |

**An Important Distinction:** Both parts use a file named `data.csv`, but they are **two different files**. Keep each part in its **own folder** so one does not overwrite the other.

---

# Part 1: Exponential Distribution (Waiting Time Between Posts)

## 1. What is this project about?

On the evening of September 26, 2026, from 6:00 PM to 10:05 PM, we observed the One Piyu Community (OPC) Facebook group and recorded the number of minutes between consecutive posts. The purpose of collecting these intervals was to determine whether the waiting time until the next post could be modeled and predicted using a statistical distribution.

To analyze the observed waiting times, we used the exponential distribution, which is commonly used to model the time between events that occur independently over a given period. In this context, the distribution allows us to examine the pattern of waiting times between posts, where shorter intervals are expected to occur more frequently while longer intervals are less common. Through this model, we assessed whether the observed posting intervals followed the expected pattern of an exponential distribution.

From our data, the project figures out:

- **how many posts show up per minute and per hour**
- **the probability** that the next post arrives within 5 or 10 minutes
- **whether our predictions are reliable**, by checking if the real waiting times look like the pattern

## 2. Why does it matter?

- **Admins know what to expect.** About how many posts come in per hour, so they can plan how many to review.
- **Unusual quiet stands out.** Since long gaps are rare, a stretch longer than 10 minutes with no posts would be unusual (only about 10.2% of waits are that long).
- **We can answer "how long until the next post?"** with actual numbers instead of guessing.

## 3. How do I open and run it?

The analysis can be performed by installing a few free software programs and following the provided steps.

### Step 1: Install the programs (one time only)

1. **R** (the engine that does the math): https://www.r-project.org/
2. **RStudio** (a friendly window for using R): https://posit.co/download/rstudio-desktop/
3. Open RStudio, then copy and paste this into the **Console** (the box at the bottom-left) and press Enter. This installs the extra tools we need.

```r
install.packages(c("rmarkdown", "knitr", "tinytex"))
tinytex::install_tinytex()
```

### Step 2: Put the files together

Save these files in the **same folder**:

| File | What it is |
|---|---|
| `FA7_Part1_Group24.Rmd` | Our analysis (the main file) |
| `data.csv` | The cleaned list of waiting times (this one is required) |
| `data_raw_with_duplicates.csv` | The original list before cleaning (optional, for reference) |

`data.csv` has four columns: `post_number`, `date`, `time`, and `interval_min`. The analysis uses `interval_min`, the minutes between a post and the one before it.

### Step 3: Run it

1. Open the `.Rmd` file in RStudio.
2. Go to **Session → Set Working Directory → To Source File Location**. *(This tells RStudio to look for the data in the same folder.)*
3. Click the **Knit** button at the top of the file window.
4. Wait a moment while the completed report is generated. It is saved as a PDF in the same folder.

## 4. What does each part do?

### Step A: Load the data

```r
data <- read.csv("data.csv")
x_obs <- data$interval_min[!is.na(data$interval_min)]
n <- length(x_obs)
```

- It opens `data.csv`, which is like opening a spreadsheet.
- It looks at the column of waiting times (`interval_min`) and **skips any blank cells**. The very first post is blank because there was no post before it to compare to.
- It counts how many waiting times we have. We kept 57 unique posts, which gives **56 waiting times**.
- Posts uploaded at the same time because of batch approval by the admins were treated as duplicates and kept only once.

### Step B: Find the average wait and the posting rate

```r
ave_interval <- mean(data$interval_min, na.rm = TRUE)
lambda <- 1 / ave_interval
per_hour <- 60 * lambda
sd_model <- 1 / lambda
sd_obs <- sd(x_obs)
```

- **Average wait:** add up all the waiting times and divide by how many there are. For our data this is **4.375 minutes**.
- **Rate (called "lambda", written λ):** how many posts arrive **per minute**. If the average wait is 2 minutes, the rate is 1 ÷ 2 = half a post per minute. This is the maximum likelihood estimate for an exponential distribution:

$$\hat{\lambda} = \frac{1}{\bar{x}}$$

- **Posts per hour:** multiply the per-minute rate by 60.
- **Standard deviation check:** for an exponential distribution, the standard deviation equals the mean ($1/\hat{\lambda}$). So we compare it to the sample standard deviation as a quick (not rigorous) sign that the model is reasonable.

### Step C: Draw the "likelihood curve" (PDF)

$$f(x) = \lambda e^{-\lambda x}, \quad x \ge 0$$

```r
x <- seq(0, 30, by = 0.1)
pdf <- lambda * exp(-lambda * x)
plot(x, pdf, type = "l")
```

- The code makes a list of waiting times from 0 to 30 minutes and works out **how likely** each one is (more precisely, the density at each point).
- It then draws a line through them.
- The curve **starts high and slopes down**, at about 0.23 at 0 minutes. This means **short waits are the most common** and long waits get less and less likely.

### Step D: Find the chances (using the area under the curve)

$$P(a \le X \le b) = \int_a^b \lambda e^{-\lambda x}\,dx = e^{-\lambda a} - e^{-\lambda b}$$

```r
p_0to5   <- exp(-lambda * 0) - exp(-lambda * 5)
p_5to10  <- exp(-lambda * 5) - exp(-lambda * 10)
p_over10 <- exp(-lambda * 10)
```

- Each line gives the **chance of a wait in a certain range**: 0 to 5 minutes, 5 to 10 minutes, or more than 10 minutes.
- These three ranges cover every possible wait, so they add up to 1.

### Step E: Find the same chances a second way (CDF)

$$F(x) = P(X \le x) = 1 - e^{-\lambda x}, \quad x \ge 0$$

```r
p5  <- 1 - exp(-lambda * 5)
p10 <- 1 - exp(-lambda * 10)
p20 <- 1 - exp(-lambda * 20)
p_over10_cdf <- 1 - p10
p5to10 <- p10 - p5
```

- This gives the **chance of waiting *up to*** 5 minutes, 10 minutes, or 20 minutes.
- For "**more than**" questions, take 100% and subtract the "up to" chance.
- It should match Step D. Getting the same answer both ways is a good double-check.

### Step F: Check if the pattern really fits

```r
hist(x_obs, breaks = seq(0, 14, by = 1), freq = FALSE)
curve(dexp(x, rate = lambda), add = TRUE, col = "red")
```

- The **blue bars** show what we actually saw: how often each waiting time happened.
- The **red line** shows what the exponential pattern predicts.
- If the red line follows the shape of the bars, then **the pattern fits our data well**. In our histogram, the bars fall from left to right and follow the shape of the red curve, which is what we expect from an exponential distribution.

## 5. What did we find?

| Result | Value |
|---|---|
| Unique posts kept | 57 |
| Waiting times | 56 |
| Observation time | 245 minutes |
| Average waiting time | 4.375 minutes |
| Rate ($\hat{\lambda}$) | 0.2286 posts per minute |
| Posts per hour | about 13.7 |
| Standard deviation under the model ($1/\hat{\lambda}$) | 4.375 minutes |
| Sample standard deviation | 3.5755 minutes |
| $P(0 \le X \le 5)$ | 0.681 |
| $P(5 \le X \le 10)$ | 0.217 |
| $P(X > 10)$ | 0.102 |
| $P(X \le 5)$ | 0.681 |
| $P(X \le 10)$ | 0.898 |

- One should expect to wait about 4.38 minutes for the next post, and about 13.7 posts show up in one hour.
- There is a 68.1% chance the next post comes within 5 minutes and an 89.8% chance it comes within 10 minutes. Gaps longer than 10 minutes are rare, at only 10.2%.
- Nearly all gaps (about 99%) are shorter than 20 minutes.
- The sample standard deviation is a bit smaller than the model's, so the data are slightly less spread out than a perfect exponential.
- For the group admins, this means the page is very active in the evening, so moderators should expect a steady stream of posts to review. A quiet stretch longer than 10 minutes would be unusual.

## 6. Limitations of the Project

- We only watched **one evening (about 4 hours)**, so other days or times might look different.
- Times were recorded in **whole minutes**, so seconds are not captured.
- We removed **batch-approved duplicates**, which makes the average gap slightly longer than it really was. Keeping them would give a higher rate and a heavier cluster near 0.
- The posting rate might **change depending on the time of the day** (for example, when admins approve posts or when people get online). The pattern assumes the rate stays steady.

---

# Part 2: Applying Normal Distribution on Campus

## 1. Overview

This project examines whether the **occupancy proportion** of selected facilities at Far Eastern University (FEU) follows a **normal distribution**. The occupancy proportion is the number of occupied tables divided by the facility's capacity:

$$\text{Occupancy Proportion} = \frac{\text{Occupied}}{\text{Capacity}}$$

A value of 0 means the space is empty, and a value of 1 means it is completely full.

**Research question:** Does campus facility occupancy rate follow a normal distribution, and what do its distribution characteristics reveal about campus space utilization?

The results are presented as a **Beamer slide presentation** (PDF), generated from an R Markdown file.

## 2. Data

Observations were collected from two locations:

| Location | Observation Period |
|---|---|
| FEU Pavilion | 10:00 to 10:30 |
| FEU Library (2nd and 3rd floors) | 12:00 to 1:00 |

The sample consists of 143 observations. Each observation is a single table area, with capacities ranging from a maximum of 12 tables to a minimum of 2 or 3.

`data.csv` must contain a column named **`Proportion`**, which holds the occupancy proportion for each observation.

## 3. Files

| File | Description |
|---|---|
| `FA7_Part2_Group24.Rmd` | Main R Markdown source (analysis code and slide content) |
| `data.csv` | Occupancy data used in the analysis |

## 4. Methods

The analysis follows these steps:

1. **Load the data** and make sure the `Proportion` column is numeric.
2. **Group the proportions into intervals** of width 0.2 (0.01 to 0.20, up to 0.81 to 1.00) and count how many observations fall in each. This is the frequency distribution table.
3. **Compute the mean and standard deviation.** The mean describes the center of the data, and the standard deviation describes how spread out it is.
4. **Compare the shape of the data to a normal curve.** The observed density curve is plotted, and a theoretical normal curve with the same mean and standard deviation is drawn over it in red. If the two curves look alike, the data is close to normal.
5. **Apply the Empirical Rule.** The percentage of observations within 1, 2, and 3 standard deviations of the mean is compared against the values expected from a normal distribution:

   | Range | Expected under a normal distribution | Observed in our data |
   |---|---|---|
   | Within 1 standard deviation | 68.3% | 59.4% |
   | Within 2 standard deviations | 95.4% | 98.6% |
   | Within 3 standard deviations | 99.7% | 100% |

6. **Measure skewness.** Skewness shows whether the data leans to one side. A value near 0 means the distribution is roughly symmetric.
7. **Detect outliers using the IQR method.** Values below $Q_1 - 1.5 \times \text{IQR}$ or above $Q_3 + 1.5 \times \text{IQR}$ are flagged as outliers, where $Q_1$ and $Q_3$ are the 25th and 75th percentiles and $\text{IQR} = Q_3 - Q_1$.
8. **Summarize the findings** and give recommendations for campus space management.

## 5. Key Findings

### Summary of results

| Statistic | Value |
|---|---|
| Sample size | 143 |
| Mean ($\mu$) | 0.6609 |
| Standard deviation ($\sigma$) | 0.2354 |
| Skewness | 0.1048 |
| $Q_1$ | 0.5 |
| $Q_3$ | 0.8542 |
| IQR | 0.3542 |
| Outlier bounds | -0.0312 to 1.3854 |
| Number of outliers | 0 |
| Within $\pm 1\sigma$ | 59.4% |
| Within $\pm 2\sigma$ | 98.6% |
| Within $\pm 3\sigma$ | 100% |

Frequency distribution table:

| Interval | Count |
|---|---|
| 0.01 to 0.20 | 2 |
| 0.21 to 0.40 | 21 |
| 0.41 to 0.60 | 42 |
| 0.61 to 0.80 | 35 |
| 0.81 to 1.00 | 43 |

### Findings

- Campus occupancy is generally **moderate to high**.
- The distribution is **approximately symmetric** (skewness = 0.1048) and has 0 outliers, so occupancy proportions are relatively balanced around the mean. However, it is **not close to a normal distribution**.
- Occupancy varies across observations, so scheduling and space allocation can be adjusted for periods of higher occupancy (e.g., exam seasons) or lower occupancy (e.g., synchronous learning days).
- The data can help the university monitor facility utilization and make informed decisions about campus space management.
- To optimize campus space, the university should implement **real-time occupancy tracking** and **dynamic scheduling** that shifts resources during peak exam seasons and utilizes low-occupancy synchronous learning days for maintenance.

## 6. Requirements

- R (4.0 or later recommended)
- RStudio (recommended)
- The `rmarkdown` and `knitr` packages
- A LaTeX installation (e.g., TinyTeX), which is required to produce Beamer slides

```r
install.packages(c("rmarkdown", "knitr", "tinytex"))
tinytex::install_tinytex()
```

Only base R functions are used in the analysis.

## 7. How to Run

1. Place `FA7_Part2_Group24.Rmd` and `data.csv` in the **same folder**.
2. Open the `.Rmd` file in RStudio.
3. Set the working directory: **Session → Set Working Directory → To Source File Location**.
4. Click **Knit** (or run `rmarkdown::render("FA7_Part2_Group24.Rmd")`).
5. The slide deck is saved as a PDF in the same folder.

## 8. Limitations

- Data was collected over **short time windows** at only **two locations**, so the results may not represent the whole campus or other times of day.
- Occupancy proportions are **bounded between 0 and 1**, while a true normal distribution extends in both directions without limit. This alone can make the fit imperfect.
- Tables have **different capacities** (from 2 or 3 up to 12), which can affect how proportions are distributed.
