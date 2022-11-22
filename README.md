---
title: "Getting and cleaning data - Assingment 1"
author: "Angel Villamizar"
date: "2022-11-22"
output:
  html_document:
    keep_md: true
    theme: cerulean
    toc: yes
    toc_depth: 1
editor_options: 
  chunk_output_type: console
---



# Introduction

The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis. You will be graded by your peers on a series of yes\no questions related to the project. You will be required to submit: 1) a tidy data set as described below, 2) a link to a Github repository with your script for performing the analysis, and 3) a code book that describes the variables, the data, and any transformations or work that you performed to clean up the data called CodeBook.md. You should also include a README.md in the repo with your scripts. This repo explains how all of the scripts work and how they are connected.

One of the most exciting areas in all of data science right now is wearable computing - see for example this article . Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. 

You should create one R script called run_analysis.R that does the following 5 steps:

# Part 1

Merges the training and the test sets to create one data set.

```r
library(data.table)

setwd(paste0("C:/Users/VillamizarA/OneDrive - Kantar/KWP/Angel/ACADEMY/COURSERA/Getting and Cleaning Data/Week 4"))

url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"

destfile <- paste0(getwd(),"/getdata_projectfiles_UCI HAR Dataset.zip")

download.file(url,destfile)

unzip(destfile)

## test data:
XTest<- read.table("UCI HAR Dataset/test/X_test.txt")
YTest<- read.table("UCI HAR Dataset/test/Y_test.txt")
SubjectTest <-read.table("UCI HAR Dataset/test/subject_test.txt")

## train data:
XTrain<- read.table("UCI HAR Dataset/train/X_train.txt")
YTrain<- read.table("UCI HAR Dataset/train/Y_train.txt")
SubjectTrain <-read.table("UCI HAR Dataset/train/subject_train.txt")

## features and activity
features<-read.table("UCI HAR Dataset/features.txt")
activity<-read.table("UCI HAR Dataset/activity_labels.txt")

##Part1 - merges train and test data in one dataset (full dataset at the end)
X<-rbind(XTest, XTrain)
Y<-rbind(YTest, YTrain)
Subject<-rbind(SubjectTest, SubjectTrain)
```

Dimension of new datasets:

```r
dim(X)
```

```
## [1] 10299   561
```

```r
dim(Y)
```

```
## [1] 10299     1
```

```r
dim(Subject)
```

```
## [1] 10299     1
```

# Part 2

Extracts only the measurements on the mean and standard deviation for each measurement.

```r
##getting features indeces which contain mean() and std() in their name
index<-grep("mean\\(\\)|std\\(\\)", features[,2]) 
length(index) ## count of features
```

```
## [1] 66
```


```r
X<-X[,index] ## getting only variables with mean\stdev
dim(X) ## checking dim of subset 
```

```
## [1] 10299    66
```

# Part 3

Uses descriptive activity names to name the activities in the data set

```r
## replacing numeric values with lookup value from activity.txt; won't reorder Y set
Y[,1]<-activity[Y[,1],2]

head(Y) 
```

```
##         V1
## 1 STANDING
## 2 STANDING
## 3 STANDING
## 4 STANDING
## 5 STANDING
## 6 STANDING
```

# Part 4

Appropriately labels the data set with descriptive variable names.

```r
## getting names for variables
names<-features[index,2] 

## updating colNames for new dataset
names(X)<-names 
names(Subject)<-"SubjectID"
names(Y)<-"Activity"
    
CleanedData<-cbind(Subject, Y, X)

## first 5 columns
head(CleanedData[,c(1:5)])
```

```
##   SubjectID Activity tBodyAcc-mean()-X tBodyAcc-mean()-Y tBodyAcc-mean()-Z
## 1         2 STANDING         0.2571778       -0.02328523       -0.01465376
## 2         2 STANDING         0.2860267       -0.01316336       -0.11908252
## 3         2 STANDING         0.2754848       -0.02605042       -0.11815167
## 4         2 STANDING         0.2702982       -0.03261387       -0.11752018
## 5         2 STANDING         0.2748330       -0.02784779       -0.12952716
## 6         2 STANDING         0.2792199       -0.01862040       -0.11390197
```

# Part 5

From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

```r
CleanedData<-data.table(CleanedData)

## features average by Subject and by activity
TidyData <- CleanedData[, lapply(.SD, mean), by = 'SubjectID,Activity']
dim(TidyData)
```

```
## [1] 180  68
```


```r
write.table(TidyData, file = "Tidy.txt", row.names = FALSE, sep = ",", quote = FALSE)
```

First 12 rows and 5 columns in Tidy dataset:

```r
head(TidyData[order(SubjectID)][,c(1:5), with = FALSE],12) 
```

```
##     SubjectID           Activity tBodyAcc-mean()-X tBodyAcc-mean()-Y
##  1:         1           STANDING         0.2789176      -0.016137590
##  2:         1            SITTING         0.2612376      -0.001308288
##  3:         1             LAYING         0.2215982      -0.040513953
##  4:         1            WALKING         0.2773308      -0.017383819
##  5:         1 WALKING_DOWNSTAIRS         0.2891883      -0.009918505
##  6:         1   WALKING_UPSTAIRS         0.2554617      -0.023953149
##  7:         2           STANDING         0.2779115      -0.018420827
##  8:         2            SITTING         0.2770874      -0.015687994
##  9:         2             LAYING         0.2813734      -0.018158740
## 10:         2            WALKING         0.2764266      -0.018594920
## 11:         2 WALKING_DOWNSTAIRS         0.2776153      -0.022661416
## 12:         2   WALKING_UPSTAIRS         0.2471648      -0.021412113
##     tBodyAcc-mean()-Z
##  1:        -0.1106018
##  2:        -0.1045442
##  3:        -0.1132036
##  4:        -0.1111481
##  5:        -0.1075662
##  6:        -0.0973020
##  7:        -0.1059085
##  8:        -0.1092183
##  9:        -0.1072456
## 10:        -0.1055004
## 11:        -0.1168129
## 12:        -0.1525139
```

