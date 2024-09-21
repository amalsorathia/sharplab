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

https://docs.google.com/spreadsheets/d/15Dq7tTdPDYBLbPMPLHwMVKY6-hIfqUhhYz9CDvSIPuY/edit?gid=0#gid=0

Attempts at gaining better accuracy for XGBoost:
1) First attempt at XGBoost comparing the four diagnoses. Attempted RandomSearchCV and GridSearchCV to choose best parameters.
   - Made sure there were no duplicates in the genes (if any genes were present for both comparisons, we dropped them because this would make accuracy worse).
   - Parameters used:
   - Accuracy obtained:
   - Logloss and Auc Curves:
   - Confusion Matrices:
  
3) Choosing top importance features.
   - choosing top 11
   - choosing top 7
   - ignoring the top importance

5) Does the covariate sex have an effect on accuracy?
   - no
   - plotted confusion matrices for sex 
  
7) Pairwise comparisons between each diagnoses. Choosing top importance features for each pairwise comparison and then using those genes as input for original XGBoost model.
   - VRFC vs TIA: 59% highest accuracy (bad logloss and auc curves)
       - 
   - VRFC vs Mimic_migraine: 73% accuracy
      - 
   - VRFC vs Mimic_no_migraine: 62.6%
      - 
   - TIA vs Mimic_migraine: 75% highest accuracy (bad logloss and auc curves)
      - 
    - TIA vs Mimic_no_migraine: 68% highest accuracy
       - 
    - Mimic_migraine vs Mimic_no_migraine: 58%
       -
    - Best Overall Accuracy: 41% (test logloss curve was still going up despite early stopping rounds being 10/15)
    - Hyperparameters: n_estimators=400,eta=.08,subsample=.7, colsample_bylevel=.5,booster="dart"
    - Test size: 0.3


9) Random sampling of top importance genes (top 5)
    - Best Overall Accuracy: 45.7%
    <img width="644" alt="Screen Shot 2024-09-18 at 9 46 59 PM" src="https://github.com/user-attachments/assets/c1356ac9-dc4d-4d9e-8764-0ec383a8791a">
    <img width="476" alt="Screen Shot 2024-09-18 at 9 47 22 PM" src="https://github.com/user-attachments/assets/27cdcc0b-05b6-46dc-8007-a1be1f43dc03">
    
    


11) Joanne's method:
    - 
        
    

12) Remove TIAs from gene sample and focus on VRFC, Mimic_migraine, Mimic_no-migraine
   - Used all genes less than 0.05 from these 3 pairwise comparisons:
     
	   - Best Accuracy: 57.9 %
	   - Best Parameters:
              XGBClassifier(base_score=None, booster='dart', callbacks=None,
              colsample_bylevel=0.5, colsample_bynode=None,
              colsample_bytree=0.7, device=None, early_stopping_rounds=None,
              enable_categorical=False, eta=0.03, eval_metric=None,
              feature_types=None, gamma=None, grow_policy=None,
              importance_type=None, interaction_constraints=None,
              learning_rate=None, max_bin=None, max_cat_threshold=None,
              max_cat_to_onehot=None, max_delta_step=None, max_depth=None,
              max_leaves=None, min_child_weight=1, missing=nan,
              monotone_constraints=None, multi_strategy=None, n_estimators=650,
              n_jobs=-1, num_parallel_tree=None, ...)
           - Important Genes:
               -
           - Confusion Matrix:
             <img width="655" alt="Screen Shot 2024-09-19 at 3 52 37 PM" src="https://github.com/user-attachments/assets/bda4384a-3989-4034-8d91-146202a0fec0">
           - Logloss & Auc Curves:
             <img width="474" alt="Screen Shot 2024-09-19 at 3 51 55 PM" src="https://github.com/user-attachments/assets/3a7a2d87-f2fd-4409-a2db-5943903f35c5">

   - Used top 20 genes from each pairwise comparisons:
           - Best Accuracy: 
	   - Best Parameters:
           - Confusion Matrix:
           - Logloss & Auc Curves:
   - Used random sampling:
          - Best Accuracy: 
	  - Best Parameters:
           - Confusion Matrix:
           - Logloss & Auc Curves:

11) Switching to Random Forest
    - Best Model: max_depth = 10, n_estimators = 100, max_features ='sqrt', min_samples_leaf = 1, min_samples_split = 2
    - Best Accuracy: 47.5 %
    - <img width="656" alt="Screen Shot 2024-09-19 at 3 42 09 PM" src="https://github.com/user-attachments/assets/01bf859f-1f89-401a-9a0b-c0eecbf9d62f">

