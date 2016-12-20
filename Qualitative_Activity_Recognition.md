# Qualitative Activity Recognition of Weight Lifting Exercises
Adham  
12/20/2016  

## Synopsis

###Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

### Goal
The goal of your project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set. You may use any of the other variables to predict with. You should create a report describing how you built your model, how you used cross validation, what you think the expected out of sample error is, and why you made the choices you did. You will also use your prediction model to predict 20 different test cases.



## Getting Data
Download the data directly using code.


```r
download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv",
              destfile = "pml-training.csv")

download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv",
              destfile = "pml-testing.csv")
```
## Read in the data
Read in the training data and the test data


```r
training = read.csv("pml-training.csv",na.strings=c("NA","#DIV/0!", ""))
testing = read.csv("pml-testing.csv",na.strings=c("NA","#DIV/0!", ""))
```

## Observe & clean the data
et a feel of the data by running str, summary, ... functions, and then sum the NAs of each variables and delete variables with so much NAs. also will delete the ID column and username to not overfit and the timestamps


```r
# see the structure of the dataset
str(training)

# see the summary of the dataset
summary(training)

# see how much NAs per column is there
colSums(is.na(training))
```


```r
# clean the data from missing values that we saw is around 19000 per some columns so we can ingore them safely

training <- training[, colSums(is.na(training)) == 0]
testing <- testing[, colSums(is.na(testing)) == 0]

training$X = NULL
training$user_name = NULL
training$raw_timestamp_part_1 = NULL
training$raw_timestamp_part_2 = NULL
training$cvtd_timestamp = NULL

testing$X = NULL
testing$user_name = NULL
testing$raw_timestamp_part_1 = NULL
testing$raw_timestamp_part_2 = NULL
testing$cvtd_timestamp = NULL
```


## Fit a model with cross validation.

We will fit different models to the data and will use Cross-Validation in all of them by passing the method = "cv" in the train control function for the trControl in Caret with folds = 5

### Start with a decision tree

```r
suppressMessages(library(caret))

set.seed(226)

model_tre <- train(
  classe~.,
  tuneLength = 1,
  data=training,
  method="rpart",
  trControl = trainControl(method="cv", number=5,verboseIter = TRUE)
  )
```

```
## Loading required package: rpart
```


```r
model_tre
```

```
## CART 
## 
## 19622 samples
##    54 predictors
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold) 
## Summary of sample sizes: 15697, 15698, 15697, 15698, 15698 
## Resampling results:
## 
##   Accuracy  Kappa    
##   0.316077  0.0482688
## 
## Tuning parameter 'cp' was held constant at a value of 0.1151545
## 
```


### Then with the Random Forest


```r
set.seed(524)
model_rf <- train(
  classe~.,
  tuneLength = 1,
  data=training,
  method="ranger",
  trControl = trainControl(method="cv", number=5,verboseIter = TRUE)
  )
```

```
## Loading required package: e1071
```

```
## Loading required package: ranger
```
Using a random forest the out of sample error would be quite small

```r
model_rf
```

```
## Random Forest 
## 
## 19622 samples
##    54 predictors
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold) 
## Summary of sample sizes: 15697, 15698, 15696, 15699, 15698 
## Resampling results:
## 
##   Accuracy  Kappa    
##   0.998522  0.9981305
## 
## Tuning parameter 'mtry' was held constant at a value of 18
## 
```



Now lets compute the confusion matrix for the winning model = Random Forest


```r
confusionMatrix(predict(model_rf, training), training$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 5580    0    0    0    0
##          B    0 3797    0    0    0
##          C    0    0 3422    0    0
##          D    0    0    0 3216    0
##          E    0    0    0    0 3607
## 
## Overall Statistics
##                                      
##                Accuracy : 1          
##                  95% CI : (0.9998, 1)
##     No Information Rate : 0.2844     
##     P-Value [Acc > NIR] : < 2.2e-16  
##                                      
##                   Kappa : 1          
##  Mcnemar's Test P-Value : NA         
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            1.0000   1.0000   1.0000   1.0000   1.0000
## Specificity            1.0000   1.0000   1.0000   1.0000   1.0000
## Pos Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Neg Pred Value         1.0000   1.0000   1.0000   1.0000   1.0000
## Prevalence             0.2844   0.1935   0.1744   0.1639   0.1838
## Detection Rate         0.2844   0.1935   0.1744   0.1639   0.1838
## Detection Prevalence   0.2844   0.1935   0.1744   0.1639   0.1838
## Balanced Accuracy      1.0000   1.0000   1.0000   1.0000   1.0000
```


And finally testing on the testing dataset

```r
predict(model_rf, testing)
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```

## Conclusions
**decision tree** (31.60%) and **random forst**(99.85%) performed better than all.

