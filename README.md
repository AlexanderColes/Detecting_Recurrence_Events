## Introduction
This repo contains the models developed in the paper titled "Evaluation of Machine Learning Methods for the Retrospective Detection of Ovarian Cancer Recurrences from Chemotherapy Data".
 Please see the paper for background on their development and results.

## Models

We developed four machine learning models commonly used in the literature. These were a Random Forest, a Decision Tree, a Conditional Inference Tree and a Logistic Regression model to identify recurrence events from chemotherapy treatment data.

The models’ final combination of candidate variables were chosen using 3 iterations of 10-fold cross-validation. The candidate models for each ML method that achieved the highest F1 score for identifying recurrence events all shared the same three candidate variables of: days between current and previous chemotherapy treatment, the Drug Regimen Group Label of that treatment and finally the Drug Regimen Group Label of the previous treatment.

## Drug Regimen Grouping

The method of recording the Drug Regimen for each treatment within a patient's EHR allows for similar drug regimens to be recorded in a variety of ways, which reduces the available context any ML method might learn from the use of that drug. To combat this, the 127 unique Drug Regimens were grouped into 26 Drug Regimen Groups by a clinician at the LTHT using their knowledge of the context each Drug Regimen is used in. The Drug Regimen and assigned Drug Regimen Group can be found in the DrugGroup.csv file. An example of the drug grouping can be seen in the below table.


|Regimen|Drug Regimen Group|
|---|---|
|CARBO 1W (C)|Carboplatin|
|CARBOPLATIN 1W (C)|Carboplatin|
|BEVA 15 maint (C)|Bevacizumab|
|BEVA 15MG/KG 6W (C)|Bevacizumab|
|BEVA_15MG/KG 6W (C)|Bevacizumab|

## Using our Models
### Loading Models
The models were developed in R studio and are saved as RDS objects. Once downloaded You can load the models into R using the following R code.
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
The Decision Tree uses the rpart package.
```R
library(rpart)
```
The Conditional Inference Tree uses the party package.
```R
library(party)
```
The Logistic Regression model is a Generalized Linear model as found in base R.

### Preprocessing Your Data for Input
The inputs to these models are the days between the current and previous chemotherapy treatment(Days.Btw.Treatments), the Drug Regimen Group Label of that treatment (Drug.Group) and finally the Drug Regimen Group Label of the previous treatment (Previous.Drug.Group).

If you have a Chemotherapy treatment table like the below example and know the date of diagnosis for the patients' cancers...

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


You will need to convert it to a table like that below, using the grouping of chemotherapy regimens, to produce the relevant inputs for the models so that they may identify recurrence events. The days since diagnosis column is used by the ProdcueSurvivalCurve script. It is not used as input for the models. 

|PatientID|Days.After.Diagnosis|Days.Btw.Treatments|Drug.Group|Previous.Drug.Group|
|---|---|---|---|---|
|A|7|0|Carboplatin|NaN|
|A|28|21|Carboplatin|Carboplatin|
|A|49|21|Carboplatin|Carboplatin|
|A|70|21|Carboplatin|Carboplatin|
|A|170|100|Bevacizumab|Carboplatin|
|A|191|21|Bevacizumab|Bevacizumab|
|B|14|0|Carboplatin/Paclitaxel|NaN|
|B|42|28|Carboplatin/Paclitaxel|Carboplatin/Paclitaxel|
|B|70|28|Carboplatin/Paclitaxel|Carboplatin/Paclitaxel|


### Predicting with the Machine Learning Models

Once you have preprocessed your data into the required input fields you can use the trained ML models to identify recurrence events in your data using the code below.
The tree-based models can be used to give a classification label of 1 = recurrence event and 0 = not a recurrence event. Below the three tree-based models are providing a classification label due to their response "type" being defined as response and class respectively.
The Logistic Regression model outputs a probability of each event being a recurrence event, the round function surrounding the predict function applies a threshold of 0.5 so that any probability greater than or equal to 0.5 is classified as a recurrence event.

```R
pred1 = predict(FinalRandomForestModel, newdata=YourDataFrame, type="response")
pred2 = predict(FinalConditionalInferenceTreeModel, newdata=YourDataFrame, type="response")
pred3 = predict(FinalDecisionTreeModel, newdata=YourDataFrame, type="class")
pred4 = round(predict(FinalLogisticRegressionModel, newdata=YourDataFrame, type="response"))
```
### Producing TTNT Survival Curves.

See the ProduceSurvivalCurves.Rmd script to produce TTNT survival curves which were used as a proxy for recurrence-free survival.





