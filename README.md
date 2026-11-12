# Pay Disparity Analysis: Base Salary by Gender, Race and Education

An end-to-end data analytics project that asks whether base salary differs across demographic groups, and whether those differences remain after controlling for job-related factors. Built with Excel, Python and Power BI.

**Quick links**

| Resource | Link |
|---|---|
| Dataset (Kaggle) | (https://bsos-data.umd.edu/tr/dataset/stem-salaries?)|
| Full project report | [[REPORT]](https://github.com/ApooorvaM/hr-analytics-pay-equity-python-statistics-powerbi/blob/main/Detailed%20report.pdf) |
| Power BI dashboard | [[DASHBOARD LINK]](https://github.com/ApooorvaM/hr-analytics-pay-equity-python-statistics-powerbi/blob/main/HR_dashboard.pbit) |
| Python notebook | [notebook/HR_notebook.ipynb](https://github.com/ApooorvaM/hr-analytics-pay-equity-python-statistics-powerbi/blob/main/HR_notebook.ipynb)|

---

## 1. Business Problem

An HR or compensation team wants to know whether pay differs by **gender, race and education**, and whether any gap is explained by legitimate factors (company, job title, experience) or remains unexplained.

**Questions**

1. How does base salary differ by gender, race and education before any controls?
2. Are the differences statistically significant?
3. Which differences remain after controlling for company, job title, years of experience and years at company?
4. How reliable is the data, and what should be fixed first?

## 2. Dataset

- **Source:** Data Science and STEM Salaries by Jack Ogozaly (Kaggle)
- **Size:** 62,642 salary submissions (2017 to 2021)
- **Unit of analysis:** a salary submission (record), not necessarily a unique employee, so results refer to records, not headcount
- **Outcome:** base salary (log-transformed in the regression models)
- **Analysis sample:** 60,338 records after removing 2,304 records with zero base salary (missing pay breakdown)

## 3. Tools

| Stage | Tool |
|---|---|
| Data preparation | Excel |
| EDA and statistics | Python (pandas, seaborn, matplotlib, SciPy, pingouin, statsmodels) |
| Dashboard | Power BI |

## 4. Data Cleaning and Quality

- Reduced 31 raw columns to 20 by dropping repetitive and indicator columns; no duplicate records found
- Standardised text, for example software engineer level variants (SDE I, SDE-II and so on). Company-specific levels (2,888 raw values) could not be unified, so years of experience was used as the seniority measure
- Kept missing gender, race and education as an explicit **"NA" category** instead of dropping or guessing values
- **Total compensation does not reconcile.** Reported total compensation differed from base + bonus + stock for **12,444 records (19.87%)**. The mean difference was $8,792.27, the median $0, with a range from -$2,328,000 to +$4,395,600. Several large gaps are consistent with stock values entered in thousands. Because of this, **base salary was used as the outcome** instead of total compensation
- **Location issues documented, not silently fixed:** city and country mismatches (for example Chennai labelled as United States) and city ID collisions

**Missing demographic data**

| Field | Missing | Share |
|---|---|---|
| Race | 40,215 | 64.20% |
| Education | 32,272 | 51.52% |
| Gender | 19,941 | 31.83% |

## 5. EDA and Feature Engineering

- **Role mix:** Software Engineers are 65.82% of records (41,231), followed by Product Managers and Engineering Managers
- **Experience level:** new `exp_level` feature from years of experience: Fresher (0 to 2), Junior (2 to 5), Mid-level (5 to 10), Senior (10 to 15), Highly Experienced (15+)
- **Reconciliation features:** `annual_salary_calc` and `salary_diff` to test total compensation against its components
- **Findings:** pay rises with experience in every role; years of experience correlates more strongly with base salary (0.374) than years at company (0.146); Asian and White records dominate every experience band, while Black, Hispanic and multiracial groups are the least represented
- **Where the nulls sit:** 58.15% of records missing demographic data are US-based Software Engineers

## 6. Statistical Analysis

**Hypothesis tests (5% significance level, Welch versions for unequal variances)**

| Variable | Test | Statistic | p-value |
|---|---|---|---|
| Gender | Welch t-test | t = 3.92 | 8.99e-05 |
| Race | Welch ANOVA | F = 171.23 | 6.90e-162 |
| Education | Welch ANOVA | F = 394.80 | 3.27e-285 |

All three are significant, but effect sizes are small (partial eta-squared of 1.5% for race and 3.4% for education), so regression was used to see which differences survive controls.

**Regression (OLS on log base salary)**

- *Unadjusted:* demographic variable only
- *Adjusted:* demographic variable + company + job title + years of experience + years at company
- Reference groups: Female, Asian, Bachelor's Degree. Coefficients are converted to percentage differences
- A robust (HC3) check on the gender model gave the same result

| Group | Unadjusted | Adjusted |
|---|---|---|
| Male vs Female | -1.02% (not significant) | **-2.39%** |
| PhD vs Bachelor's | +52.78% | **+35.85%** |
| Master's vs Bachelor's | +24.52% | **+15.51%** |
| Some College vs Bachelor's | +19.88% | **+4.60%** |
| Highschool vs Bachelor's | +3.00% (not significant) | +3.73% (p = 0.075, not significant) |
| Hispanic vs Asian | +8.05% | **+5.56%** |
| White vs Asian | +14.33% | **+9.82%** |
| Black vs Asian | +7.93% | **+9.00%** |
| Two or More vs Asian | +13.99% | **+10.97%** |

Adding job-related controls lifts R² from 0.6% to 4.2% (demographics only) to about 44% to 46%, so most explainable pay variation comes from role, company and experience, not demographics.

## 7. Dashboard (Power BI)

A two-page interactive dashboard with a demographic selector (education, gender, race).

**Page 1: Compensation Overview** (records, average and median base salary, filters by year and job title)

<img width="452" height="251" alt="first_page" src="https://github.com/user-attachments/assets/258edd13-68bb-4728-ace2-791d5c292d24" />


**Page 2: What explains pay differences?** (unadjusted vs adjusted effects, regression results, methodology, key highlights)

<img width="445" height="250" alt="second_page" src="https://github.com/user-attachments/assets/f92e3f5f-2cc0-44a4-b672-4ed09ade922d" />


## 8. Key Findings

1. **Education has the largest adjusted effect:** PhD +35.85% and Master's +15.51% versus Bachelor's.
2. **Controls shrink raw gaps substantially** (for example PhD from 52.78% to 35.85%), so raw comparisons overstate differences.
3. **The adjusted gender gap is small:** male records are about 2.39% lower than female records.
4. **Every disclosed race group is higher than Asian after controls** (+5.56% to +10.97%).
5. **Most demographic data is missing,** which limits how far any finding can be generalised.

## 9. Business Recommendations

1. **Fix demographic data collection first:** make the fields part of the standard form, separate "prefer not to say" from "not collected", and explain how the data is used.
2. **Standardise compensation reporting:** separate base, bonus and stock fields, unit and total validation checks, and a recorded currency and period.
3. **Run adjusted pay reviews, not raw comparisons:** compare people in similar roles, companies and experience bands.
4. **Investigate the race pattern against the Asian reference group:** review leveling and role placement before taking any pay action.
5. **Monitor the small gender gap over time** with the same controls, and re-test as demographic coverage improves.
6. **Review how degrees are rewarded:** decide whether degree premiums should be built into pay bands or reflected only through level and role.
7. **Improve data from small groups** (Black, Hispanic and multiracial records are each under 2% of the data) so future estimates are more reliable.

## 10. Limitations and Future Scope

**Limitations**

- Heavy, non-random missing demographic data (records with undisclosed values have the highest average pay)
- Sample dominated by US Software Engineers; self-reported, voluntary submissions
- Records, not people; results show association, not causation or discrimination
- Company levels could not be standardised, and location is not controlled in the demographic models (company and nation overlap)
- Reported total compensation does not reconcile, so only base salary was modelled
- Non-normal residuals; HC3 checks completed for the gender model only

**Future scope**

- Formal treatment of missing data (imputation or weighting as sensitivity checks)
- Add location with company fixed effects, test interactions, and test intersectional effects
- Complete HC3 checks for race and education models
- Repair location data and unit errors, rebuild a reconciled total compensation measure, then repeat for bonus and stock
- Extend the dashboard with confidence intervals, a time view and drill-through


## Author

**Apoorva Maheshwari**
https://www.linkedin.com/in/m-apoorva-05b1b2b8/
