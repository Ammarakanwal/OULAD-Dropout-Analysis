
EDTECH ANALYTICS PROJECT
Student Dropout Prediction
& Risk Analysis Report


Platforms Used:  Google Colab  |  Microsoft Power BI

Dataset: Open University Learning Analytics Dataset (OULAD)

Analysis Phase
Python / Google Colab	Visualization Phase
Microsoft Power BI

Date: June 2026

1. Project Overview


This report documents a comprehensive end-to-end EdTech analytics project focused on predicting student dropout and classifying learners by risk level. The project leverages the Open University Learning Analytics Dataset (OULAD) — one of the most detailed publicly available educational datasets — and progresses through data engineering, machine learning model development, and business intelligence dashboarding.

The workflow was executed across two industry-standard platforms: Google Colab for data processing and machine learning, and Microsoft Power BI for interactive reporting and stakeholder-facing dashboards.

1.1 Project Objectives
•Predict whether a student will drop out of a course using machine learning
•Classify students into risk tiers: Low, Medium, and High dropout risk
•Engineer meaningful behavioral and academic features from raw interaction data
•Build an interactive Power BI dashboard for educational administrators
•Enable data-driven early intervention to improve student retention

1.2 Platforms & Tools
Platform	Role	Key Libraries / Features	Output
Google Colab	Data Engineering & ML	pandas, numpy, scikit-learn, matplotlib, seaborn	Trained Model + Risk CSV
Microsoft Power BI	Dashboard & Reporting	Custom DAX measures, Slicers, Charts	3-Page Interactive Dashboard

2. Dataset Description


The project uses the Open University Learning Analytics Dataset (OULAD), which captures student activity across online courses offered by the UK's Open University. The dataset includes 7 relational tables representing student demographics, course structures, assessment records, and virtual learning environment (VLE) interactions.

2.1 Dataset Tables
Table	Description	Key Columns
studentInfo	Core demographics and final result per student-course	id_student, gender, region, highest_education, age_band, disability, final_result
studentVle	Raw VLE interaction logs (10.6M rows)	id_student, id_site, date, sum_click
studentAssessment	Assessment submission scores	id_student, id_assessment, score, is_banked
studentRegistration	Course registration and withdrawal dates	id_student, code_module, date_registration, date_unregistration
assessments	Assessment metadata and deadlines	id_assessment, code_module, assessment_type, date, weight
courses	Course presentation details and length	code_module, code_presentation, module_presentation_length
vle	VLE resource metadata (activity types)	id_site, code_module, activity_type, week_from, week_to

Notable scale: the studentVle table alone contains over 10.6 million rows, making it the richest source of behavioral engagement signals in the dataset.

3. Data Analysis — Google Colab


All data processing, feature engineering, and machine learning tasks were performed in Google Colab using Python. The workflow followed a structured pipeline from raw data ingestion through to model evaluation and risk scoring output.

3.1 Environment Setup
The notebook was executed on Google Colab, a cloud-hosted Jupyter environment. Files were uploaded interactively using the Colab files API and the following core libraries were imported:

Library	Purpose
pandas	Data loading, manipulation, merging, and aggregation
numpy	Numerical operations and array handling
scikit-learn	Machine learning models, train/test split, evaluation metrics
matplotlib	Custom visualizations (bar charts, ROC curves)
seaborn	Statistical heatmaps (confusion matrix, classification report)

3.2 Data Loading & Initial Exploration
All 7 CSV files were loaded into individual pandas DataFrames. Initial exploration tasks included:
•Previewing the first rows of each table using .head()
•Inspecting data types and non-null counts using .info()
•Replacing '?' placeholder values with NaN across all tables
•Converting date and numeric columns (stored as strings) to correct numeric types using pd.to_numeric(..., errors='coerce')

Columns converted included: assessments.date, studentAssessment.score, studentRegistration.date_registration, studentRegistration.date_unregistration, vle.week_from, and vle.week_to.

3.3 Feature Aggregation
3.3.1 VLE Engagement Summary
Because the studentVle table contains 10.6 million individual interaction rows, it was aggregated by student-course-presentation grouping to create engagement features:
•total_clicks — Total number of clicks across all VLE resources
•avg_clicks — Mean clicks per interaction session
•total_interactions — Count of interaction records

3.3.2 Assessment Performance Summary
Student assessment records were aggregated at the student level to extract academic performance features:
•avg_score — Mean score across all assessments
•total_assessments — Number of assessments submitted
•highest_score — Maximum score achieved
•lowest_score — Minimum score recorded

3.4 Master Dataset Construction
A denormalized analytical base table (master_df) was built by performing a sequence of left joins:

Step	Source Table	Features Added
1	studentInfo (base)	gender, region, highest_education, age_band, disability, final_result
2	assessment_summary	avg_score, highest_score, lowest_score, total_assessments
3	student_vle_summary	total_clicks, avg_clicks, total_interactions
4	registration_summary	date_registration, date_unregistration

3.5 Feature Engineering
After merging, new derived features were engineered to improve model signal:

Feature	Formula / Logic	Purpose
dropout_flag	1 if date_unregistration is not null	Binary target variable for ML
study_duration	date_unregistration - date_registration	Time active in course
engagement_score	total_clicks / (total_interactions + 1)	Clicks per interaction (intensity)
performance_level	High (≥70), Medium (≥40), Low (<40)	Categorical academic tier

3.6 Machine Learning Models
3.6.1 Data Preparation
•Target variable: dropout_flag (binary classification)
•Missing numeric values filled using column median
•Missing categorical values (imd_band) filled with 'Unknown'
•One-hot encoding applied to all categorical columns using pd.get_dummies()
•Train/test split: 80% train, 20% test with stratified sampling (random_state=42)

3.6.2 Model 1 — Logistic Regression
An initial baseline Logistic Regression model was trained (max_iter=1000) to establish a performance benchmark using linear decision boundaries.

3.6.3 Model 2 — Random Forest Classifier
A Random Forest Classifier was trained as the primary model (n_estimators=200, class_weight='balanced', random_state=42). The balanced class weighting addressed the class imbalance in the dropout_flag target.

3.7 Model Evaluation & Results
Metric	Logistic Regression	Random Forest
Accuracy	Baseline	84.5%
ROC AUC	—	High (see ROC curve)
Class Weight	Default	Balanced
Estimators	N/A	200 trees

3.8 Visualizations Produced in Colab
The following charts were generated in-notebook and later embedded in the Power BI dashboard:

•Feature Importance Bar Chart — Top 10 features ranked by Random Forest importance
•Confusion Matrix Heatmap — Actual vs. Predicted dropout classifications
•Classification Report Heatmap — Precision, Recall, F1 by class
•ROC Curve — Random Forest ROC with AUC score
•Precision/Recall/F1 Bar Chart — Side-by-side metric comparison by class

3.9 Risk Scoring & Output Export
After model evaluation, dropout probabilities were generated for all test-set students using predict_proba(). Each student was assigned a risk tier:
•Low Risk — Dropout probability < 0.30
•Medium Risk — Dropout probability 0.30–0.70
•High Risk — Dropout probability > 0.70

Two output CSV files were exported for Power BI consumption:
•student_dropout_risk_results.csv — Student-level predictions with probability and risk tier
•feature_importance.csv — Feature importance rankings from the Random Forest model

4. Dashboard Development — Microsoft Power BI


A three-page interactive Power BI dashboard was built using the exported CSV outputs from Google Colab combined with the master_df analytical dataset. The dashboard is designed for educational administrators and analysts to monitor dropout risk, engagement patterns, and model performance.

4.1 Data Sources Connected
•master_table — Full analytical dataset with demographics, engagement, and academic features
•student_dropout_risk_results — ML model output with dropout probability and risk level per student
•ModelMetrics — Custom static table holding model performance KPI values

Relationships were established between the tables on id_student, code_module, and code_presentation keys for cross-table filtering.

4.2 Dashboard Pages

Page 1 — Risk Level Overview
The primary landing page for executive and administrative users. Provides a high-level snapshot of dropout risk across the student population.

Visual Type	Title / Metric	Description
KPI Card	Total Students	Count of all students in the dataset
KPI Card	Average Score %	Mean assessment score across all students
KPI Card	Dropout Ratio %	Percentage of students who withdrew
KPI Card	Total Dropout	Absolute count of dropout events
KPI Card	High Risk Students	Students classified as high dropout risk by ML model
Donut Chart	Final Result Distribution	Breakdown by Pass, Fail, Withdrawn, Distinction
Donut Chart	Dropout Rate by Gender	Male vs. Female dropout comparison
Donut Chart	Risk Distribution	Low / Medium / High risk tier proportions from ML model
Column Chart	Top Modules with Highest Dropout	Course module dropout counts ranked
Matrix Table	Risk Summary	Risk level vs. Pass Rate % and Dropout Rate % breakdown
Slicers (x5)	Gender, Region, Education, Disability, Age Band	Interactive filters for all page visuals

Page 2 — Student Engagement Analysis
An analytical page focused on learning behavior patterns and the relationship between engagement and dropout risk.

Visual Type	Title / Metric	Description
KPI Card	Avg VLE Clicks	Average total VLE clicks per student
KPI Card	Avg Interactions	Average total interaction count per student
KPI Card	Avg Engagement Score	Mean engineered engagement intensity score
KPI Card	Avg Study Days	Mean study duration (days active in course)
Clustered Column	Avg Clicks by Risk Level	Click volumes segmented by Low/Medium/High risk
Clustered Column	Avg Score by Risk Level	Academic performance segmented by performance_level
Scatter Chart	Engagement vs. Performance	avg_score vs. total_clicks coloured by risk level
Data Table	Top Engaged Students	Student ID, clicks, avg_score, and risk level ranked

Page 3 — Model Performance Metrics
A technical page summarizing the accuracy and reliability of the Random Forest machine learning model, intended for data science and analytics audiences.

Visual Type	Title / Metric	Description
KPI Card	Accuracy	Overall model accuracy on the test set
KPI Card	Precision	Precision score from classification report
KPI Card	Recall	Recall score — dropout class detection rate
KPI Card	F1 Score	Harmonic mean of precision and recall
KPI Card	ROC AUC	Area under the ROC curve
Image Visual	Confusion Matrix	Seaborn heatmap from Colab embedded as image
Image Visual	ROC Curve	ROC plot with AUC score from Colab embedded as image
Image Visual	Feature Importance Bar Chart	Top dropout predictors from Random Forest
Data Table	Top High-Risk Students	Student IDs with dropout probability and risk level

4.3 DAX Measures & Calculated Columns
Custom DAX measures were created in Power BI to power the KPI cards and matrix table:
•Total Students — COUNTROWS(master_table)
•Dropout Rate % — DIVIDE(SUM(dropout_flag), COUNTROWS()) * 100
•Pass Rate % — Percentage of students with final_result = 'Pass'
•High Risk Students — CALCULATE(COUNTROWS, risk_level = 'High Risk')
•Avg Score — AVERAGE(avg_score) formatted as percentage

5. Key Insights & Findings


5.1 Model Performance
•The Random Forest model achieved 84.5% accuracy in predicting student dropout, significantly outperforming the Logistic Regression baseline
•The balanced class_weight setting was critical to improving detection of the minority dropout class
•The ROC AUC score indicates strong discriminative ability between dropout and non-dropout students

5.2 Top Dropout Predictors
Feature importance analysis from the Random Forest revealed the most influential predictors of dropout:
•study_duration — How long the student remained enrolled before withdrawal
•total_clicks and engagement_score — VLE interaction volume and intensity
•avg_score — Academic performance is a strong dropout signal
•date_registration — Later registration correlates with higher dropout risk
•age_band and highest_education — Demographic factors contribute to risk

5.3 Engagement Patterns
•High-risk students show significantly lower VLE click volumes compared to Low-risk students
•A positive correlation exists between total_clicks and avg_score, visualized in the Engagement vs. Performance scatter plot
•Students with engagement_score below threshold are disproportionately represented in the High Risk tier

5.4 Risk Distribution
The ML model's risk scoring system classified the test-set student population into three actionable tiers enabling targeted early intervention by educational administrators.

6. Visualizations


The following charts were generated in Google Colab and are also embedded in the Power BI Model Performance page:

6.1 Feature Importance — Top Dropout Predictors
Horizontal bar chart showing the top 10 features ranked by their importance score in the Random Forest model:



6.2 Confusion Matrix
Seaborn heatmap comparing actual vs. predicted dropout classifications on the test set:



6.3 ROC Curve
Receiver Operating Characteristic curve for the Random Forest model showing the AUC score:



7. Project Summary


This project delivered a complete EdTech analytics pipeline covering every stage from raw data ingestion to an interactive executive dashboard. The following table summarizes all tasks completed on each platform:

7.1 Google Colab — Task Summary
#	Task	Detail
1	Data Ingestion	Loaded 7 CSV tables from OULAD into pandas DataFrames
2	Data Exploration	Used .head(), .info(), .shape() on all tables
3	Data Cleaning	Replaced '?' with NaN; converted dtypes; handled nulls
4	VLE Aggregation	Summarized 10.6M rows → total_clicks, avg_clicks, total_interactions
5	Assessment Aggregation	Built avg_score, highest_score, lowest_score, total_assessments
6	Master Dataset Build	Merged 4 tables into one analytical base table via left joins
7	Feature Engineering	Created dropout_flag, study_duration, engagement_score, performance_level
8	ML Preprocessing	One-hot encoding, median imputation, train/test split (80/20 stratified)
9	Logistic Regression	Baseline model trained and evaluated
10	Random Forest	200-tree classifier with balanced weights; 84.5% accuracy achieved
11	Model Evaluation	Accuracy, Precision, Recall, F1, ROC AUC; confusion matrix
12	Risk Scoring	Probability-based Low/Medium/High risk tier assignment for each student
13	Visualization	Feature importance, confusion matrix, ROC curve, classification heatmap
14	CSV Export	Exported student_dropout_risk_results.csv and feature_importance.csv

7.2 Microsoft Power BI — Task Summary
#	Task	Detail
1	Data Import	Loaded master_table and risk results CSVs into Power BI
2	Data Modelling	Defined table relationships on id_student, code_module keys
3	DAX Measures	Created custom KPIs: Dropout Rate %, Pass Rate %, High Risk Students
4	Page 1: Risk Overview	5 KPI cards, 3 donut charts, 1 column chart, pivot matrix, 5 slicers
5	Page 2: Engagement	4 KPI cards, 2 clustered columns, scatter chart, top students table
6	Page 3: Model Metrics	5 metric KPI cards, 3 embedded ML images, high-risk students table
7	Interactivity	Cross-filtering slicers for gender, region, education, disability, age
8	Image Embedding	Confusion matrix, ROC curve, and feature importance chart embedded



End of Report
