# **Predicting Survival in Cirrhosis Patients Using R - A Statistical Analysis with Kaplan-Meier and Cox Models**

---

## **Objective**

This analysis aims to uncover clinical and demographic predictors of survival in patients with cirrhosis using the Mayo Clinic PBC dataset. It walks through a complete survival analysis pipeline using:

- **Kaplan-Meier survival curves** for exploratory visualization  
- **Cox Proportional Hazards Models** enhanced with natural splines  
- An interactive **Shiny app** to predict 3-year survival probability

---

## **Dataset**

- **Source**: Mayo Clinic Trial on Primary Biliary Cirrhosis (PBC)  
- **Sample Size**: 418 patients (focus on 312 randomized trial participants with complete records)  
- **Link to the Dataset**: [Kaggle - Cirrhosis Dataset](https://www.kaggle.com/datasets/fedesoriano/cirrhosis-prediction-dataset)

---

## **Analysis Workflow**

### **1. Data Preparation**

The dataset was loaded into R and pre-processed by renaming the outcome variable (`Status`) to a binary form (`Event`), where death = 1 and censored = 0. Only randomized trial participants were retained due to data completeness.

### **2. Missing Data Handling**

Missing values in continuous variables like cholesterol, copper, triglycerides, and platelets were imputed using **median imputation**, a robust method less sensitive to outliers. This ensures a consistent dataset without introducing skewed estimates.

### **3. Kaplan-Meier Survival Curves**

Using the `survival` and `survminer` packages, Kaplan-Meier curves were constructed to compare survival probabilities across groups. These were stratified by variables like:

- **Sex** (Male vs Female)  
- **Ascites** (Presence vs Absence)  
- **Edema**, **Spiders**, **Stage**, **Age Quartiles**

---

## **Kaplan-Meier Survival Analysis Results**

| **Stratifying Variable** | **Significance (p-value)** | **Interpretation** |
|--------------------------|----------------------------|---------------------|
| Overall                  | Not specified              | Shows a steady decline in survival probability over time for the overall population. |
| Age Quartile            | < 0.0001                   | Higher age quartiles (especially Q4) are associated with significantly lower survival probabilities. |
| Ascites                  | < 0.0001                   | Presence of ascites is strongly associated with much lower survival compared to absence of ascites. |
| Drug                     | 0.75                       | No significant difference in survival between D-penicillamine and placebo groups. |
| Edema                    | < 0.0001                   | Patients with more severe edema (especially 'Y') have much lower survival probabilities. |
| Hepatomegaly             | < 0.0001                   | Presence of hepatomegaly is associated with significantly reduced survival. |
| Sex                      | 0.039                      | Males have lower survival compared to females, though the difference is modest. |
| Spiders                  | < 0.0001                   | Presence of spider angiomas is associated with poorer survival outcomes. |
| Stage                    | < 0.0001                   | Survival decreases progressively with increasing disease stage; Stage 4 has the worst prognosis. |

The Kaplan-Meier curves and the Log-rank test demonstrate that clinical signs of liver decompensation (ascites, edema, hepatomegaly, spiders), age, disease stage, and to a lesser extent, sex, significantly affect survival. Meanwhile, treatment with D-penicillamine does not appear to provide a survival advantage.

---

### **4. Linearity Check with Martingale Residuals**

Before modeling continuous predictors in a Cox model, **Martingale residual plots** were used to assess linearity. This revealed **non-linear relationships** for variables like:

- Bilirubin  
- SGOT  
- Prothrombin  
- Copper  
- Alkaline Phosphatase

These variables were therefore modeled using **natural splines** in the Cox model.

---

### **5. Cox Proportional Hazards Model with Natural Splines**

A multivariable Cox model was fit using the `coxph()` function, incorporating:

- Log-transformed Bilirubin  
- Albumin (linear)  
- Splines for Prothrombin, SGOT, Alk_Phos, and Copper  
- Categorical variables like Sex, Stage, Edema, Age Group, etc.

---

### **Cox Proportional Hazards Model Interpretation**

| Variable               | HR (exp(coef)) | 95% CI                                  | p-value    | Interpretation                                                      |
|------------------------|----------------|------------------------------------------|------------|----------------------------------------------------------------------|
| log(Bilirubin + 0.1)   | 1.85351        | [1.37729, 2.4944]                        | 4.65e-05   | Higher bilirubin significantly increases hazard (risk of death).    |
| Albumin                | 0.46392        | [0.26831, 0.8021]                        | 0.00597    | Higher albumin is protective (reduces hazard).                      |
| Prothrombin (ns df=3)  | Varies         | [0.80216, 12.3117] to [0.70537, 158.6838]| 0.10027 to 0.08776 | Nonlinear effect; some components approach significance.         |
| SGOT (ns df=3)         | Varies         | [0.93438, 9.2169] to [0.32925, 40.7989]  | 0.06522 to 0.29079 | Nonlinear effect; marginal or non-significant.               |
| Alk_Phos (ns df=3)     | Varies         | [0.42283, 7.1572] to [0.05477, 2.0348]   | 0.44296 to 0.23418 | Nonlinear and non-significant overall.                        |
| Copper (ns df=3)       | Varies         | [1.01529, 44.2950] to [0.18133, 8.1219]  | 0.04724 to 0.84181 | Component 1 significant: higher copper increases hazard.       |
| Sex (Male)             | 1.19359        | [0.67892, 2.0984]                        | 0.53873    | No significant difference between sexes.                          |
| Ascites (Yes)          | 1.52021        | [0.80732, 2.8626]                        | 0.19459    | Ascites increases risk but not statistically significant.         |
| Hepatomegaly (Yes)     | 1.05586        | [0.64242, 1.7354]                        | 0.83022    | Hepatomegaly not significantly associated with survival.           |
| Spiders (Yes)          | 0.98261        | [0.62674, 1.5406]                        | 0.93906    | Spiders not significantly associated with survival.                |
| Edema (Slight)         | 1.25468        | [0.68497, 2.2982]                        | 0.46253    | Slight edema not significant.                                      |
| Edema (Marked)         | 2.23512        | [1.14035, 4.3809]                        | 0.01916    | Marked edema significantly increases hazard.                       |
| Stage                  | 1.32272        | [0.96872, 1.8061]                        | 0.07841    | Higher stage marginally increases hazard.                          |
| Age Quartile 2         | 0.91496        | [0.48253, 1.7349]                        | 0.78544    | No significant difference vs Q1.                                   |
| Age Quartile 3         | 2.16064        | [1.18371, 3.9438]                        | 0.0121     | Q3 has significantly higher hazard than Q1.                        |
| Age Quartile 4         | 2.08186        | [1.11973, 3.8707]                        | 0.02048    | Q4 has significantly higher hazard than Q1.                        |

---

### **6. 3-Year Survival Probability Prediction Tool**

After fitting a Cox Proportional Hazards Model to the dataset, a custom R-based tool was developed to estimate the 3-year survival probability for individual patients based on significant clinical features. This prediction utility uses the coefficients derived from the final Cox model and applies them in a risk calculation without needing to re-train the model each time.

#### **User Inputs:**

- Bilirubin level (mg/dL)  
- Albumin level (g/dL)  
- Edema status: `"N"` for none, `"S"` for slight, `"Y"` for marked  
- Age quartile: `1` to `4` based on the patient’s age

#### **Model Logic:**

The **Linear Predictor (LP)** is calculated as:

```
LP = (β₁ × log(bilirubin + 0.1)) + (β₂ × albumin) + (β₃ × edema) + (β₄ × age quartile)
```

Then the **3-year survival probability** is computed as:

```
Ŝ(t = 3 years) = S₀(t) ^ exp(LP)
```

Where `S₀(t)` is the **baseline survival** at 3 years, estimated as `0.75`.

---

### **7. Shiny App: Real-Time 3-Year Survival Prediction**

An interactive Shiny app was built to allow users to input patient characteristics and receive a **predicted 3-year survival probability** based on the fitted Cox model.

#### **Inputs include:**

- Bilirubin (mg/dL)  
- Albumin (g/dL)  
- Disease Stage (1–4)  
- Age Quartile Group (Q1–Q4)

The app calculates the linear predictor and estimates survival probability using the model’s baseline survival rate and patient-specific risk.

---

## **8. Applications**

- **Clinical Use**: Estimate patient-specific prognosis in cirrhosis  
- **Research Use**: Explore model-driven risk stratification for clinical trials

---

## **Contact**

Questions? Reach out via [LinkedIn](https://www.linkedin.com/in/eswarkamisetti)  
📧 [eswarkamisetti@gmail.com](mailto:eswarkamisetti@gmail.com)
