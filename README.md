
## 📌 Project Overview
LoS.png
Hospital readmissions within 30 days of discharge present massive financial penalties for healthcare networks under the Hospital Readmissions Reduction Program (HRRP) and often flag vital opportunities for improved transitional care. 

This repository delivers an end-to-end data analytics portfolio piece. It uses complex SQL transformations to clean and model chronic patient data, coupled with a dynamic Power BI Executive Dashboard designed to isolate risk factors driving prolonged hospital stays and rapid patient returns.

## 📊 Business Objectives
* **Identify High-Risk Segments:** Pinpoint primary diagnoses and comorbidity loads highly correlated with 30-day readmissions.
* **Optimize Capacity Management:** Analyze Length of Stay (LOS) trends across age brackets to improve hospital bed turnover.
* **Clinical Risk Mitigation:** Stratify patients into actionable risk tiers to help discharge planners allocate home health care resources efficiently.

## 📁 Repository Structure
WITH ClinicalCohort AS (
    SELECT 
        encounter_id,
        patient_nbr AS patient_id,
        age,
        time_in_hospital AS length_of_stay,
        num_medications,
        number_diagnoses AS comorbidity_count,
        diag_1 AS primary_diagnosis_code,
        -- Normalize the readmission flag to a binary integer for Power BI calculation
        CASE 
            WHEN readmitted = '<30' THEN 1 
            ELSE 0 
        END AS is_30_day_readmit,
        -- Order patient encounters chronologically using window functions
        ROW_NUMBER() OVER (PARTITION BY patient_nbr ORDER BY encounter_id) as encounter_sequence
    FROM hospital_admissions
)
SELECT 
    encounter_id,
    patient_id,
    age,
    length_of_stay,
    num_medications,
    comorbidity_count,
    is_30_day_readmit,
    encounter_sequence,
    -- Label high-utilizer chronic patients (More than 5 documented diagnoses)
    CASE 
        WHEN comorbidity_count > 5 THEN 'High Risk'
        WHEN comorbidity_count BETWEEN 3 AND 5 THEN 'Moderate Risk'
        ELSE 'Low Risk'
    END AS clinical_risk_tier
FROM ClinicalCohort;


