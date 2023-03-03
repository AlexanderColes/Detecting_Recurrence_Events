# Evaluation of Machine Learning Methods for the Detection of Multiple Cancer Recurrences from Chemotherapy Data


## Introduction
Cancer recurrences are poorly recorded within electronic health records around the World, which hinders effective research into the efficiency of cancer treatments.
Here we present the final ML models from "paper" that identify ovarian cancer patients treatment events as either a recurrence event or not. 

Our models were trained on patients who were diagnosed at the Leeds Teaching Hospitals Trust (LTHT). The entire data set comprised 531 fully curated patients, with 6,619 recorded chemotherapy treatments, including 127 unique Drug Regimens, spanning a time from 2008 to 2021. A manual chart review had retrospectively augmented  treatment events with a respective programme number. Within the LTHT EHR, a programme number identifies the instance of progression or recurrence of each patient’s cancer that each chemotherapy treatment was being used to treat. This data set can then be further divided, by analysing the programme numbers, into the two subgroups: patients whose cancer had not recurred (n=258, ~49%), and those whose cancer did recur (n=273,~51%). The trainining set consisted of a uniformly random sampled 354  patients (66.66%) and the final test set contained the other 177 patients (33.33%).

The unique labelling of every chemotherapy treatment event with a programme number, which can be converted to a binary label indicating a recurrence event coinciding with a change in programme number, facilitates a different method to those found in existing literature to identify the date of recurrence. Our models are able to use supervised learning to classify each treatment event a patient had as either a recurrence event or not.

We define a recurrence event as a proxy chemotherapy treatment event subsequent to a recurrence or progression diagnosis aligning to the initial labelling system used by the manual chart reviewers.The clinical difference between a recurrence and a progression is a contested subject. We refer to the Cancer Outcomes and Services Dataset’s (COSD) definition in that a recurrence of a patient’s cancer is defined as “the return of cancer after treatment and after a period of time during which the cancer cannot be detected” and is only differentiated from the progression of their cancer due to the patient having “previously been informed that they are free of the disease or that the disease is not detectable”25 Therefore, in our research, there is little interest in discriminating between the two outcomes of recurrence or progression.

## Models

We developed four machine learning models commonly used in the literature. These were, a Random Forest, a Decision Tree, a Conditional Inference Tree and a Logistic Regression model to identify recureceneventsfrom chemotherapy treatment data.

The models final combinationn of candidate variables were chose using 10 fold cross validation. The candidate models for each ML method that achived the highest F1 score for identifying recurrence evenst all shared the same three candidate variables of: days between current and previous chemotherapy treatment, the Drug Regimen Group Label of that treatement and finaly the Drug Regimen Group Label of the previous treatment.

## Drug Regimen Grouping

The method of recording the Drug Regimen for each treatment within a patients EHR allows for the similar drug regimens to be recorded in a variety of ways, which reduces the available context any ML method might learn from the use of that drug. To combat this, the 127 unique Drug Regimens were grouped into 26 Drug Regimen Groups by a clinician at the LTHT using their knowledge of the context each Drug Regimen is used in. The Drug Regimen and assigned Drug Regimen Group can be found in the DrugGroup.csv file. An exmaple of the drug grouping can be seen in the below table.

|Regimen|Drug Regimen Group|
|---|---|
|CARBO 1W (C)|Carboplatin|
|CARBOPLATIN 1W (C)|Carboplatin|
|BEVA 15 maint (C)|Bevacizumab|
|BEVA 15MG/KG 6W (C)|Bevacizumab|
|BEVA_15MG/KG 6W (C)|Bevacizumab|

## Using our Models
### Loading Models
The models were developed in R studio and are saved as RDS objetcs. Once downloaded You can load the models into R using the following R code.
``` R
FinalRandomForestModel <- readRDS("FinalRandomForestModel.rds")
FinalConditionalInferenceTreeModel <- readRDS("FinalConditionalInferenceTreeModel.rds")
FinalDecisionTreeModel <- readRDS("FinalDecisionTreeModel.rds")
FinalLogisticRegressionModel<- readRDS("FinalLogisticRegressionModel.rds")
```

The Random Forest uses the randomForest package.
```R
library(randomForest)
```
The Decision Tree useus the rpart package.
```R
library(rpart)
```
The Conditional Inference Tree uses the party package.
```R
library(party)
```
The Logistic Regression model is a Generlised Linear model as found in base R.

### Preprocessing Your Data for Input
The inputs to these models are the three vraibels descibed ealrier. These are the days between current and previous chemotherapy treatment(Days.Btw.Treatments), the Drug Regimen Group Label of that treatement (Drug.Group) and finaly the Drug Regimen Group Label of the previous treatment (Previous.Drug.Group).

IF you have a Chemotherapy treatment table like the below example.

|PatientID|Chemotherapy.Cycle.Start.Date|Regimen.Label|
|---|---|---|
|A|01/01/2023|CARBO (AUC) 21D (C)|
|A|22/01/2023|CARBO (AUC) 21D (C)|
|A|12/02/2023|CARBO 21D (C)|
|A|05/03/2023|CARBO 21D (C)|
|A|13/06/2023|BEVA 15 maint (C)|
|A|04/07/2023|BEVA 15 maint (C)|
|B|01/02/2023|PACLITAX/CARBO (C)|
|B|01/03/2023|PACLITAX/CARBO (C)|
|B|29/03/2023|PACLITAX/CARBO (C)|


You will need to convert it to a table like that below with the relevant inputs for the models so that they may identify recurrence events.

|PatientID|Days.Btw.Treatments|Drug.Group|Previous.Drug.Group|
|---|---|---|---|
|A|0|Carboplatin|NaN|
|A|21|Carboplatin|Carboplatin|
|A|21|Carboplatin|Carboplatin|
|A|21|Carboplatin|Carboplatin|
|A|100|Bevacizumab|Carboplatin|
|A|21|Bevacizumab|Bevacizumab|
|B|0|Carboplatin/Paclitaxel|NaN|
|B|28|Carboplatin/Paclitaxel|Carboplatin/Paclitaxel|
|B|28|Carboplatin/Paclitaxel|Carboplatin/Paclitaxel|


### Predicting with the Machine Learning Models

Once you have preprocessed your data into the required input fields you can use the trained ML models to identfiy recurrence events in your data using the code below.
The tree based models can be used to give a probablity of a recurrence event for each event or a classification label of 1 = recurrence event and 0 = not a recurrence event. Below the three tree based models are providing a classification label due to their repsonse "type" being defined as repsonse and class respectively.
The Logistic Rgeression models outputs a probability of each event being a recurrence event, the round function surrounding the predict function applies a threshold of 0.5 so that any probabilty greater than or equal to 0.5 is classified as a recurrence event.

```R
pred1 = predict(FinalRandomForestModel, newdata=YourDataFrame, type="response")
pred2 = predict(FinalConditionalInferenceTreeModel, newdata=YourDataFrame, type="response")
pred3 = predict(FinalDecisionTreeModel, newdata=YourDataFrame, type="class")
pred4 = round(predict(FinalLogisticRegressionModel, newdata=YourDataFrame, type="response"))
```






