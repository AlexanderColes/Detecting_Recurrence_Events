# Evaluation of Machine Learning Methods for the Detection of Multiple Cancer Recurrences from Chemotherapy Data
Evaluation of multiple ML models to detect multiple recurrence events in ovarian cancer patients from structured chemotherapy timeseries data.

## Introduction
Cancer recurrences are poorly recorded within electronic health records around the World, which hinders effective research into the efficiency of cancer treatments.
Here we present the final ML models from "paper" that identify ovarian cancer patients treatment events as either a recurrence event or not. 

Our models were trained on patients who were diagnosed at the Leeds Teaching Hospitals Trust (LTHT). The entire data set comprised 531 fully curated patients, with 6,619 recorded chemotherapy treatments, including 127 unique Drug Regimens, spanning a time from 2008 to 2021. A manual chart review had retrospectively augmented  treatment events with a respective programme number. Within the LTHT EHR, a programme number identifies the instance of progression or recurrence of each patient’s cancer that each chemotherapy treatment was being used to treat. This data set can then be further divided, by analysing the programme numbers, into the two subgroups: patients whose cancer had not recurred (n=258, ~49%), and those whose cancer did recur (n=273,~51%). The trainining set consisted of a uniformly random sampled 354 patients and the final test set contained the other 177 patients.

The unique labelling of every chemotherapy treatment event with a programme number, which can be converted to a binary label indicating a recurrence event coinciding with a change in programme number, facilitates a different method to those found in existing literature to identify the date of recurrence. Our models are able to use supervised learning to classify each treatment event a patient had as either a recurrence event or not.

We define a recurrence event as a proxy chemotherapy treatment event subsequent to a recurrence or progression diagnosis aligning to the initial labelling system used by the manual chart reviewers.The clinical difference between a recurrence and a progression is a contested subject. We refer to the Cancer Outcomes and Services Dataset’s (COSD) definition in that a recurrence of a patient’s cancer is defined as “the return of cancer after treatment and after a period of time during which the cancer cannot be detected” and is only differentiated from the progression of their cancer due to the patient having “previously been informed that they are free of the disease or that the disease is not detectable”25 Therefore, in our research, there is little interest in discriminating between the two outcomes of recurrence or progression.

## Models

We developed four machine learning models commonly used in the literature. These were, a Random Forest, a Decision Tree, a Conditional Inference Tree and a Logistic Regression model.

The models final combinationn of candidate variables were chose using 10 fold cross validation. The candiate models for each ML method that achived the highest F1 score for identifying recurrence evenst all shared the same three candidate variables of: days between current and previous chemotherapy treatment, the Drug Regimen Group Label of that treatement and finaly the Drug Regimen Group Label of the previous treatment.

## Drug Regimen Grouping

The method of recording the Drug Regimen for each treatment within a patients EHR allows for the similar drug regimens to be recorded in a variety of ways, which reduces the available context any ML method might learn from the use of that drug. To combat this, the 127 unique Drug Regimens were grouped into 26 Drug Regimen Groups by a clinician at the LTHT using their knowledge of the context each Drug Regimen is used in. The Drug Regimen and assigned Drug Regimen Group can be found in the DrugGroup.csv file. An exmaple of the drug grouping can be seen in the below table.

|Regimen|Drug Regimen Group|
|---|---|
|CARBO (AUC) 21D (C)|Carboplatin|
|CARBO 21D (C)|Carboplatin
|BEVA 15 maint (C)|Bevacizumab|
|BEVA 15MG/KG 6W (C)|Bevacizumab|

## Using our Models

The models were developed in R sudio and are are as RDS objetcs. Yoi can load teh models into R using 
``` R
FinalRandomForestModel <- readRDS("FinalRandomForestModel.rds")
```









