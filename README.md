# sharplab

Purpose: 

Transient ischemic attacks (TIAs) occur when there is a disruption in blood supply to the brain. This can often be due to blood clots, cerebral ischemia, or more. The symptoms of a TIA are very similar to that of a strokes and TIAs are often called mini-strokes. A TIA usually requires emergency care and often indicates possible future health problems such as a stroke. If a TIA is not treated, the risk of an actual stroke occuring in the next 90 days is very high - especially in the next two days after the initial onset of TIA.

One problem, however, is TIAs are often incorrectly diagnosed since other medical ailments such as migraines, hypoglycemia, and seizures can have very similar TIA symptoms. Although the symptoms are similar for these medical ailments, an accurate diagnosis for a TIA is critical to the patient's health and recovery. This is especially true if the patient is brough to the emergency department and the doctors have to figure out a diagnosis.

To help doctors accurately make diagnoses for TIA, Sharp Lab is interested in seeing if patients' gene expressions are able to indicate whether or not a patient is likely to be experiencing a TIA compared to another neurological medical ailment.

To do this, we are using patient blood samples to develop a machine learning model that can help us predict if a patient having certain genes' expressed indicates a TIA diagnosis.

Patient Cohort:

- Number of patients:
- Diagnosis groups:
    - VRFC (control group)
    - True TIA (patients who were actually having a TIA)
    - Mimic_migraine (patients who had a migraine but their symptoms mimicked a TIA)
    - Mimic_no_migraine (patients who did not have a migraine but some other medical ailment but their symptoms mimicked a TIA)
- Covariates:
  - Sex (recorded as gender in raw data)
  - Hypertension_history 
  - Hypercholesterolemia
  - Diabetes
  - Age

Procedure:

To prepare the raw data we used dplyr and tidyverse. This included a list of patient samples with their health history and their gene expressions for a multitude of different genes. We focused only on the covariate history for each patient.

Then, we used the MatchIt package to balance the covariates across all diagnosis groups. Then, we split the balanced diagnosis groups into derivation (65% of samples) and validation cohorts (35% of samples). Then, we ran MatchIt again to ensure that validation and derivation cohorts of each diagnosis were also balanced. This was to ensure that any differentially expressed gene we see is due to the diagnosis and not any other covariates.

After that, we used DESEQ2 in order to discover differentially expressed genes (DEGs) between patients with different diagnoses. It was ran for each pairwise comparison for both cohorts: (VRFC vs True TIA), (VRFC vs Mimic_migraine), (VRFC vs Mimic_no_migraine), (True TIA vs Mimic_migraine), (True TIA vs Mimic_no_migraine), (Mimic_migraine vs Mimic_no_migraine).

The criteria we used was that the unadjusted p-value was less than 0.05 and the fold change was greater than or equal to 1.2. We wanted to genes that were the top 20 overlapping genes for both validation and derivation. If we did not get 20 overlapping genes, we filled them in using the derivation cohort genes. Moreover, in case we got 0 for some of the aforementioned criterias, we pulled from unadjusted p-values that may be greater than 0.05 but were still the smallest possible. Thus, we ended up with 20 genes for each pariwise comparison per cohort (so a total of 360 genes).

Lastly, we used XGBoost to develop a machine learning model in order to predict a True TIA based on the expression of certain genes. The training group was the validation cohort samples and the testing group was the derivation cohort samples.
