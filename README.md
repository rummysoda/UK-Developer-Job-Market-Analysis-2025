# UK Developer Job Market Analysis 2025

## Overview

This project analyses the UK software development job market using data from the [Stack Overflow Developer Survey 2025](https://survey.stackoverflow.co/2025/), which received over 49,000 responses from 177 countries. The dataset was filtered to UK-based employed and self-employed developers only, giving approximately 1,700 respondents. Salaries are reported in GBP.

The analysis explores salary distributions, how pay varies by job role, years of experience and education level, the most used technologies, and which skills command the highest salaries in the UK market.

A key finding is that legacy languages like Elixir and COBOL are associated with the highest salaries. There is also a striking pay gap between UK and US Data Scientists, with the US national median at $152k (~£115k) compared to roughly £57k in the UK.

## The Questions
1. What does the salary distribution look like for UK developers?
2. How does salary vary by job role and years of experience?
3. Which industries pay UK developers the most?
4. How does education level affect salary in the UK?
5. Which programming languages, databases and platforms are most used by UK developers?
6. Which languages are associated with the highest salaries in the UK?

## Tools Used
- **Pandas** for data cleaning, filtering and grouping
- **Matplotlib** for histograms, box plots, bar charts and scatter plots
- **Seaborn** for gradient bar charts and regression scatter plots
- **NumPy**

## Data Preparation & Cleaning
The raw survey CSV was loaded and filtered to UK employed and self-employed developers only. The `ConvertedCompYearly` column (pre-converted to USD by Stack Overflow) was multiplied by **0.77** to convert to GBP. Outliers were removed using the **IQR method**. Irrelevant `DevType` values (`"Other (please specify):"` and `"Student"`) were replaced with NaN then dropped.

columns such as `LanguageHaveWorkedWith` store responses as semicolon-separated strings (e.g. `"Python;SQL;JavaScript"`). These were split and exploded into individual rows before any counting or grouping.

```python
# Filter to UK employed developers
df = df[ (df['Country'] == 'United Kingdom of Great Britain and Northern Ireland') &((df['Employment'] == 'Employed') | (df['Employment'] == 'Independent contractor, freelancer, or self-employed'))]

# Select relevant columns only
df = df[[
        'Age',
        'EdLevel',
        'WorkExp',
        'DevType',
        'Industry',
        'LanguageHaveWorkedWith',
        'DatabaseHaveWorkedWith',
        'PlatformHaveWorkedWith',
        'WebframeHaveWorkedWith',
        'ConvertedCompYearly'
]]

# Convert salary to GBP
salaries['ConvertedCompYearly'] = salaries['ConvertedCompYearly'] * 0.77

# Get rid of outliers using IQR
q1 = salaries.ConvertedCompYearly.quantile(0.25)
q3 = salaries.ConvertedCompYearly.quantile(0.75)
iqr = q3 - q1
lowerFence = q1 - 1.5 * iqr
upperFence = q3 + 1.5 * iqr
  
salaries = salaries[(salaries.ConvertedCompYearly < upperFence) & (salaries.ConvertedCompYearly > lowerFence) ]
```

## The Analysis
### 1. Salary Distribution

**Chart:** Histogram + boxplot of individual salaries

The distribution is right-skewed where most UK developers earn between £40k–£75k with a median around £68k. The box plot underneath the histogram shows the IQR and a small number of high earners above the upper fence.

  <img width="1189" height="790" alt="image" src="https://github.com/user-attachments/assets/845636b2-83d3-4897-95af-cfe759885b62" />


### 2. Salary by Role

**Charts:** Boxplot sorted by median + gradient horizontal bar chart

Salary values for each role were grouped into lists using `groupby` + `apply(list)`, then sorted by median using `reindex`. Product Managers and Senior Executives earn the most, while Support Engineers and Game Developers sit at the lower end.

  <img width="1302" height="1014" alt="image" src="https://github.com/user-attachments/assets/746c6707-b79f-4ac8-a0e8-bd69a86f46a5" />


### 3. Salary by Industry

**Chart:** Horizontal Gradient bar chart of median salary per industry

Fintech and Banking/Financial Services pay the most for UK developers, consistent with London's position as a global financial hub. Higher Education and Government sit at the bottom.

  <img width="1304" height="1014" alt="image" src="https://github.com/user-attachments/assets/958a312b-fb17-4d55-a463-c822550b831f" />


### 4. Salary by Years of Experience

**Charts:** Boxplot sorted by median + scatter plot with regression line

A scatter plot with `sns.regplot()` shows a positive trend where salary increases with experience but with high variance at every level, which suggest role and industry matter more than experience alone. `x_jitter=0.3` was added to spread overlapping dots since `WorkExp` is stored as whole numbers.

  <img width="1415" height="783" alt="image" src="https://github.com/user-attachments/assets/157dbd0e-e25d-4146-82e9-6cb9e8e821bc" />


### 5. Salary by Degree

**Charts:** Box plot + gradient bar chart

`"Other (please specify):"` and `"Primary/elementary school"` were excluded. Master's degree holders earn the highest median salary though the gap between degree levels is smaller than expected.

  <img width="1675" height="783" alt="image" src="https://github.com/user-attachments/assets/66b0d010-08d9-43ec-a2af-678a6f0697e7" />


### 6. Top Languages, Databases & Platforms

**Charts:** Horizontal bar charts

Multi-select columns were split on `";"`, exploded and counted with `value_counts()`. Python, SQL and JavaScript are the most used languages. PostgreSQL and MySQL lead for databases, AWS and Docker for platforms.

### 7. Highest Paying Languages

**Chart:** Horizontal Gradient bar chart of median salary per language

Languages were exploded and merged with salary data, then grouped by language to get median salary. Elixir, Mojo, Lisp and COBOL are associated with the highest median salaries which are niche languages used in industries where there is no room for small errors due to the critical nature of their systems such as banking systems and aerospace.

  <img width="1208" height="783" alt="image" src="https://github.com/user-attachments/assets/bdce9133-5501-4d8f-8db8-f3390207037f" />


## Key Insights
- **Legacy languages pay the most**: Elixir and COBOL developers earn significantly above the UK median.
- **Fintech leads all industries**: UK developers in Fintech earn notably more than those in Software Development
- **A Master's degree helps but isn't essential** : Salary differences between education levels are smaller than expected.
- **The UK vs US pay gap is significant**: Data Scientist roles in the US have a national median of $152k, with cities like San Francisco reaching $178k. UK Data Scientists earn around £57k median. This gap is larger than most other roles when comparing the two markets.
- 
  <img width="1200" height="729" alt="image" src="https://github.com/user-attachments/assets/61dfd37a-e737-4c10-bfce-a7e1cc7d9f08" />

  >**Source: U.S. Bureau of Labor Statistics** 

## Challenges Faced
- **Multi-select columns** columns like `LanguageHaveWorkedWith` store multiple values as semicolon-separated strings, requiring `str.split(';')`, `.explode()` and `.str.strip()` before analysis.
- **Sorting grouped data** — box plots required lists sorted by median using `reindex(apply(np.median).sort_values().index)`, while bar charts used `sort_values()` directly on a median Series
