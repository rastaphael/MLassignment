---
title: "ML assignment"
author: "Raphael Villedieu"
date: "October 24, 2017"
output: html_document
---



## R Markdown

Load test and training sets with appropriate NA strings


```r
trainingSet <- read.csv("pml-training.csv", na.strings=c("NA","#DIV/0!",""))
testingSet <- read.csv("pml-testing.csv", na.strings=c("NA","#DIV/0!",""))
```

Remove IDs because they are irrelevant


```r
training <- trainingSet[c(-2)]
testing <- testingSet[c(-2)]
```


```r
for (i in 1:length(testing) ) {
        for(j in 1:length(training)) {
        if(  names(testing[i]) == names( training[j]) )  {
            class(testing[[i]]) <- class(training[[j]])
        }      
    }      
}
```
The whole training set is used for training. However we are going to use cross-validation for evaluating our model.
Create 10 training and testing set from the training set.


```r
library(caret)
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```
## Want to understand how all the pieces fit together? Buy the
## ggplot2 book: http://ggplot2.org/book/
```

```r
trainingFolds <- createFolds(training[,1], k = 10, list = TRUE, returnTrain = TRUE)
```

Train a decision tree with rpart on the whole training set


```r
library(rpart)
modfit <- rpart(classe ~ ., data=training, method="class")
```

Plot the relative error:

```r
printcp(modfit)
```

```
## 
## Classification tree:
## rpart(formula = classe ~ ., data = training, method = "class")
## 
## Variables actually used in tree construction:
## [1] X
## 
## Root node error: 14042/19622 = 0.71563
## 
## n= 19622 
## 
##        CP nsplit rel error     xerror       xstd
## 1 0.27040      0   1.00000 1.00000000 0.00450019
## 2 0.25687      1   0.72960 0.72966814 0.00498294
## 3 0.24370      2   0.47272 0.47286711 0.00472013
## 4 0.22903      3   0.22903 0.22916963 0.00369375
## 5 0.01000      4   0.00000 0.00021364 0.00012334
```

```r
predictions <- predict(modfit, training, type = "class")
```

The accuracy is:


```r
confusionMatrix(predictions, training$classe)$overall[[1]]
```

```
## [1] 1
```

Now repeat training and evaluating using the 10-folds:


```r
accuracy <- vector(length = 10)
for(i in 1:10){
  # Get i-th fold and build training and testing sets
  XTrain = training[ trainingFolds[[i]] , ]
  XTest = training[ - trainingFolds[[i]] , ]
  # Train model with decision trees
  Xmodfit <- rpart(classe ~ ., data=XTrain, method="class")
  # Predict using current model for unseen data (XTest)
  predictions <- predict(Xmodfit, XTest, type = "class")
  accuracy[i] <- confusionMatrix(predictions, XTest$classe)$overall[[1]]
}  
```

The accuracies for the 10 models on unseen data are:


```r
accuracy
```

```
##  [1] 0.9994903 0.9994903 1.0000000 0.9994903 1.0000000 1.0000000 1.0000000
##  [8] 1.0000000 0.9994901 0.9994906
```

The mean is:


```r
mean(accuracy)
```

```
## [1] 0.9997452
```

Now we can use the model we train on the whole training set in order to predict the class of the test set.
The predictions are:

```r
predict(modfit, testing, type = "class")
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  A  A  A  A  A  A  A  A  A  A  A  A  A  A  A  A  A  A  A  A 
## Levels: A B C D E
```




