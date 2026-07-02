# 🏥 Hospital Bed & Resource Utilization Analysis
### Python EDA Project | Pandas | Matplotlib | Seaborn | Jupyter Notebook

---

## 📌 Project Overview

This project analyzes **100,000 patient hospital admission records** across 5 facilities to identify patterns in bed utilization, length of stay, and readmission behavior. The goal is to help hospital administrators understand where bed capacity is being strained and what drives longer patient stays.

Built entirely in Python using Pandas for data manipulation and Seaborn/Matplotlib for visualization.

---

## 🎯 Business Problem

A hospital network wants to understand:
- How long are patients staying on average, and is there variation across facilities?
- Which patient factors are most strongly associated with longer stays?
- Which facilities are under the most bed pressure?
- Which patients should be flagged as high resource-use cases?

---

## 📊 Dataset

**Hospital Length of Stay Dataset — Microsoft**
- Source: Kaggle (originally released as a Microsoft healthcare analytics case study)
- Size: **100,000 patient records × 28 columns**
- Facilities: 5 hospital facilities (A, B, C, D, E)
- Date range: January 2012 — January 2013
- Key columns: `lengthofstay`, `rcount`, `facid`, `vdate`, `discharged`, `bmi`, `pulse`, `bloodureanitro`, and 11 medical condition flags

---

## 🔍 Project Stages

### Stage 1 — Data Loading & First Look
- Loaded dataset using Pandas
- Inspected shape (100,000 × 28), column names, and data types
- Identified 5 string columns needing attention: `vdate`, `discharged`, `rcount`, `gender`, `facid`
- Checked unique values in categorical columns before any cleaning

### Stage 2 — Data Cleaning
Three specific cleaning steps performed:

**1. rcount conversion**
`rcount` was stored as text with values `'0','1','2','3','4','5+'`. Replaced `'5+'` with `'5'` then converted to integer. Decision rationale: exact counts beyond 5 were not material to the analysis.

**2. Date conversion**
Converted `vdate` and `discharged` from string (`M/D/YYYY` format) to proper `datetime64` using `pd.to_datetime()` — enabling date arithmetic for validation.

**3. Data validation**
Calculated actual length of stay from date difference `(discharged - vdate).dt.days` and compared against the existing `lengthofstay` column. **Confirmed 100% match across all 100,000 records** before proceeding with analysis.

---

### Stage 3 — Descriptive Statistics
Ran `df.describe()` to understand distributions of all numeric columns. Key observations:
- `lengthofstay`: mean 4.0 days, range 1-17 days
- `bloodureanitro`: max value of 682 flagged as extreme outlier (severe kidney failure case)
- `bmi`: mean 29.8 — patient population skews overweight
- `rcount`: mean 1.11 prior readmissions per patient

---

### Stage 4 — Visualizations & Analysis

#### Plot 1 — Length of Stay Distribution
**Chart type:** Histogram + Boxplot

**Finding:** Right-skewed bimodal distribution — two distinct patient populations visible:
- Peak 1 at 1-2 days (quick turnaround patients)
- Peak 2 at 4-5 days (medium complexity patients)
- 15,828 patients (15.8%) stayed beyond 7 days — these are the high bed-utilization cases
- 5 statistical outliers between 13-17 days

---

#### Plot 2 — Average Length of Stay by Facility
**Chart type:** Bar chart

**Finding:** Clear two-tier pattern across facilities:
```
Facility A → 3.27 days  (low utilization)
Facility B → 3.28 days  (low utilization)
Facility C → 4.89 days  (high utilization)
Facility D → 4.83 days  (high utilization)
Facility E → 5.16 days  (highest utilization)
```
Facilities C, D, E average ~57% longer stays than A and B.

---

#### Plot 3 — Patient Count by Facility
**Chart type:** Bar chart

**Finding:**
```
Facility A → 30,035 patients
Facility B → 30,012 patients
Facility C →  4,699 patients
Facility D →  4,499 patients
Facility E → 30,755 patients
```
Facility E has the highest patient volume AND the longest average stay — the primary bed pressure point. Facilities C and D have low volume but long stays, suggesting specialization in complex cases.

---

#### Plot 4 — Readmission Count vs Length of Stay
**Chart type:** Bar chart

**Finding:** Perfect monotonic increasing relationship:
```
0 readmissions → 2.72 days avg stay
1 readmission  → 3.70 days avg stay
2 readmissions → 5.27 days avg stay
3 readmissions → 6.27 days avg stay
4 readmissions → 7.26 days avg stay
5 readmissions → 8.29 days avg stay
```
Each additional readmission adds ~1.11 days to average stay. Patients with 5 readmissions stay **3x longer** than first-time patients.

---

#### Plot 5 — Correlation Heatmap
**Chart type:** Seaborn heatmap with annotation

**Key correlations with lengthofstay:**
```
rcount         → 0.75  (strong positive)
bloodureanitro → 0.15  (weak positive)
All vitals     → ~0.00 (no meaningful relationship)
```
`rcount` is overwhelmingly the strongest predictor of length of stay. Clinical measurements at admission (BMI, pulse, glucose, sodium) show near-zero correlation with LOS.

---

### Stage 5 — High Risk Patient Segmentation

**Criteria (derived from analysis findings):**
- 3+ prior readmissions (`rcount >= 3`)
- Length of stay 7+ days (`lengthofstay >= 7`)
- Elevated blood urea nitrogen (`bloodureanitro > 20`)

**Results:**
```
Total high risk patients : 1,681 (1.68% of all patients)

By facility:
Facility A → 153 high risk patients
Facility B → 145 high risk patients
Facility C → 172 high risk patients
Facility D →  82 high risk patients
Facility E → 1,129 high risk patients
```

---

## 💡 Key Insights

**Insight 1 — Readmission history dominates everything else**
Prior readmission count (rcount) has a 0.75 correlation with length of stay — stronger than any clinical measurement. Patient history, not current clinical status, is the primary driver of bed utilization.

**Insight 2 — Facility E is the critical pressure point**
Highest patient volume (30,755) + longest average stay (5.16 days) + most high-risk patients (1,129) = maximum bed strain. Any bed capacity intervention should prioritize Facility E.

**Insight 3 — Two distinct patient populations exist**
The bimodal LOS distribution reveals quick-turnaround patients (1-2 days) and medium-complexity patients (4-5 days). Managing these groups with separate protocols would be more effective than a one-size-fits-all approach.

**Insight 4 — Clinical vitals at admission are poor predictors**
BMI, pulse, sodium, and glucose all show near-zero correlation with LOS. Admission screening based on these values alone is insufficient for predicting resource utilization.

---

## 📋 Recommendations

**1. Readmission Prevention Program**
Patients with 3+ readmissions stay 3x longer per visit. Invest in post-discharge follow-up, medication adherence support, and outpatient care coordination to break the readmission cycle.

**2. Facility E Capacity Planning**
Review staffing ratios, bed allocation, and discharge planning resources specifically for Facility E — the network's highest-pressure location.

**3. High Risk Patient Pathway**
Flag patients with 3+ prior readmissions at admission for early specialist involvement and proactive discharge planning — before their stay extends.

**4. Facilities C & D Specialization Review**
Low volume but consistently long stays suggests these facilities handle complex cases. Ensure adequate specialist availability and that they are not absorbing overflow from A, B, E.

---

## 🛠️ Tools & Libraries Used

| Tool | Purpose |
|---|---|
| Python 3 | Core programming language |
| Pandas | Data loading, cleaning, manipulation, groupby |
| NumPy | Numerical operations |
| Matplotlib | Base plotting library |
| Seaborn | Statistical visualizations |
| Jupyter Notebook | Interactive development environment |

---

## 💡 Key Python Concepts Demonstrated

| Concept | Where Used |
|---|---|
| `pd.read_csv()` | Loading dataset |
| `df.info()`, `df.describe()` | Data profiling |
| `.unique()` | Checking categorical values |
| `.replace()` + `.astype()` | Type cleaning with edge case handling |
| `pd.to_datetime()` | Date conversion |
| `.dt.days` | Date arithmetic |
| `df.groupby().mean()` | Aggregation (equivalent to SQL GROUP BY) |
| `reset_index()` | Converting Series to DataFrame |
| Boolean indexing | Filtering rows by conditions |
| `sns.histplot()` | Distribution visualization |
| `sns.boxplot()` | Spread and outlier visualization |
| `sns.barplot()` | Category comparison |
| `sns.heatmap()` | Correlation matrix visualization |
| `.corr()` | Correlation calculation |
| f-strings | Formatted print output |
| `plt.subplot()` | Multiple charts in one figure |

---

## 📁 Project Structure

```
hospital-bed-utilization-project/
│
├── README.md
├── hospital_bed_utilization_analysis.ipynb   -- Full Jupyter notebook
└── LengthOfStay.csv                          -- Dataset (download from Kaggle)
```

---

## 🚀 How to Run This Project

1. Download the dataset from [Kaggle](https://www.kaggle.com/datasets/aayushchou/hospital-length-of-stay-dataset-microsoft)
2. Place `LengthOfStay.csv` in the project folder
3. Install required libraries:
```
pip install pandas numpy matplotlib seaborn jupyter
```
4. Open `hospital_bed_utilization_analysis.ipynb` in Jupyter Notebook
5. Run all cells in order from top to bottom

---

## 👤 Author

**Deepesh A**
B.E Computer Science Engineering | Semester 5 | Batch 2022
Sri Venkateswara College of Engineering, Chennai

📧 deepesh.a@svce.ac.in

---

*This project was built as part of a Data Analyst portfolio to demonstrate Python EDA skills including data cleaning, exploratory analysis, visualization, and business insight generation on a real-world healthcare dataset.*
