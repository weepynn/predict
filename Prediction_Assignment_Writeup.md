---
title: "Prediction Assignment Writeup"
author: "Lim Wee Pynn"
date: "7 June 2018"
output: 
  html_document: 
    keep_md: yes
---



## Objective
The goal of your project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set. The "classe" variable is the Unilateral Dumbbell Biceps Curl in five different fashions: exactly according to the specification (Class A), throwing the elbows to the front (Class B), lifting the dumbbell only halfway (Class C), lowering the dumbbell only halfway (Class D) and throwing the hips to the front (Class E). Hence, based on all the variables in the data set, you wish to predict the outcomes of the unilateral Dumbbell Biceps Curl. 

This report is summarized into 4 parts. 1) Data import, cleaning and partitioning. 2) Data Prediction and Modelling, 3) Validation of model 4) Testing the model with the "pml-testing" data set.

## 1) Data import, cleaning and partitioning

Firstly, load up the data sets. I will remove all the irrelevant variables for our prediction model. They are the first 7 variables of the data sets. Then, I will remove all the columns with 'NA' values from both the test and training sets. 


```r
x <- read.csv("pml-training.csv", na.strings = c("NA", "", "#DIV0!"))
y <- read.csv("pml-testing.csv", na.strings = c("NA", "", "#DIV0!"))
dim(x)
```

```
## [1] 19622   160
```

```r
dim(y)
```

```
## [1]  20 160
```

```r
head(colnames(x), 7)
```

```
## [1] "X"                    "user_name"            "raw_timestamp_part_1"
## [4] "raw_timestamp_part_2" "cvtd_timestamp"       "new_window"          
## [7] "num_window"
```

```r
head(colnames(y), 7)
```

```
## [1] "X"                    "user_name"            "raw_timestamp_part_1"
## [4] "raw_timestamp_part_2" "cvtd_timestamp"       "new_window"          
## [7] "num_window"
```

```r
train <- x[,8:dim(x)[2]]
test <- y[,8:dim(y)[2]]
train <- train[, colSums(is.na(train)) == 0]
test <- test[, colSums(is.na(test))==0]
dim(train)
```

```
## [1] 19622    53
```

```r
dim(test)
```

```
## [1] 20 53
```
Now, we are reduced to 53 variables including the "classe" variable in both sets. Next, I will split the training set into two sets, 70% for training, and 30% for testing. 

```r
library(caret)
```

```
## Loading required package: lattice
```

```
## Warning: package 'lattice' was built under R version 3.3.3
```

```
## Loading required package: ggplot2
```

```
## Warning: package 'ggplot2' was built under R version 3.3.3
```

```r
set.seed(12345)
inTrain <- createDataPartition(train$classe, p = 0.7, list = F)
training <- train[inTrain,]
testing <- train[-inTrain,]
dim(training)
```

```
## [1] 13737    53
```

```r
dim(testing)
```

```
## [1] 5885   53
```
## 2) Data Prediction and Modelling

The algorithm used here is Random Forest. 


```r
library(e1071)
```

```
## Warning: package 'e1071' was built under R version 3.3.3
```

```r
setting <- trainControl(method ="cv", 5)
RandForest <- train(classe ~., data = training, method = 'rf', trControl = setting)
RandForest
```

```
## Random Forest 
## 
## 13737 samples
##    52 predictor
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold) 
## Summary of sample sizes: 10990, 10989, 10990, 10989, 10990 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy   Kappa    
##    2    0.9911916  0.9888574
##   27    0.9900268  0.9873839
##   52    0.9842756  0.9801059
## 
## Accuracy was used to select the optimal model using the largest value.
## The final value used for the model was mtry = 2.
```
Next, test the model on the out-of-sample partitioned test set. 

## 3) Validation of the model

```r
prediction <- predict(RandForest, testing)
confusionMatrix(testing$classe, prediction)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1673    1    0    0    0
##          B   13 1120    6    0    0
##          C    0   14 1009    3    0
##          D    0    0   23  941    0
##          E    0    0    0    3 1079
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9893          
##                  95% CI : (0.9863, 0.9918)
##     No Information Rate : 0.2865          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9865          
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9923   0.9868   0.9721   0.9937   1.0000
## Specificity            0.9998   0.9960   0.9965   0.9953   0.9994
## Pos Pred Value         0.9994   0.9833   0.9834   0.9761   0.9972
## Neg Pred Value         0.9969   0.9968   0.9940   0.9988   1.0000
## Prevalence             0.2865   0.1929   0.1764   0.1609   0.1833
## Detection Rate         0.2843   0.1903   0.1715   0.1599   0.1833
## Detection Prevalence   0.2845   0.1935   0.1743   0.1638   0.1839
## Balanced Accuracy      0.9960   0.9914   0.9843   0.9945   0.9997
```
Model has an accuracy of about 98%. So, we shall use this model to predict the "pml-testing" data set.

## 4) Testing the model with the "pml-testing" data set


```r
predict20 <- predict(RandForest, test)
predict20
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```
