---
title: "Getting and Cleaning Data"
author: "Hao Gong"
date: "November 20, 2017"
output: html_document
---

```{r}
library(data.table)
```

Import Data
```{r}

projectPath <-"C:\\Users\\Hao\\OneDrive\\Project\\UCI HAR Dataset\\"
setwd(projectPath)
#Train Dataset
data_train_x <- data.table(read.table('./train/X_train.txt'))
data_train_y_sub <- data.table(read.table('./train/subject_train.txt'))
data_train_y_act <- data.table(read.table('./train/y_train.txt'))
data_train_y <- cbind(data_train_y_sub, data_train_y_act)
```


```{r}
data_test_X <- data.table(read.table('./test/X_test.txt'))
data_test_y_sub <- data.table(read.table('./test/subject_test.txt'))
data_test_y_act <- data.table(read.table('./test/Y_test.txt'))
data_test_y <- cbind(data_test_y_sub, data_test_y_act)
```

```{r}
head(data_train_x)
head(data_train_y)
dim(data_train_x)
dim(data_train_y)
```

```{r}
data_train <- cbind(data_train_y, data_train_x)
data_test <- cbind(data_test_y, data_test_X)
#names
names(data_train)[1] <- 'Subject'
names(data_test)[1] <- 'Subject'
names(data_train)[2] <- 'ActivityNum'
names(data_test)[2] <- 'ActivityNum'
head(data_train)
head(data_test)
```


Merge train and test 
```{r}
data_all <- rbind(data_train, data_test)
dim(data_all)
tail(data_all)
```



Extracts only the measurements on the mean and standard deviation for each measurement.
```{r}
colnames_get <- fread('./features.txt')
names(colnames_get) <- c('featureNum', 'featureCode')
colnames_get$featureNum <- colnames_get[,paste0('V',featureNum)]
colnames_get
result_name <- colnames_get[grep('\\mean|\\-std', featureCode)]

result_1 <- data_all[,c('Subject','ActivityNum',result_name$featureNum), with = FALSE]
names(result_1) <- c('Subject','ActivityNum',result_name$featureCode)

names(result_1)<-gsub('-','_',names(result_1))

names(result_1)<-gsub('[()]','', names(result_1))

```


```{r}
activities <- fread('./activity_labels.txt')
setnames(activities, names(activities), c("ActivityNum", "ActivityName"))

```

Merge
```{r}
data_final <- merge(activities, result_1, by = 'ActivityNum')
```


```{r}
data_final_1<- data_final
setkey(data_final_1, Subject, ActivityName)
tidy_table <- data_final_1[,list(count = .N, average= mean(tBodyAcc_mean_X)), by = key(data_final_1)]
```


```{r}
getwd()
write.table(tidy_table,file = file.path(getwd(), 'submit.txt'))
write.table(data_final,file = file.path(getwd(), 'tidy_data.txt'))
```

