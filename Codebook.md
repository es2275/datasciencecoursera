## Loading the data

if(!file.exists("./data")){dir.create("./data")}

# link given in the course
fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"

# Downlading the file
download.file(fileUrl,destfile="./data/Dataset.zip",method="curl")

# Unzipping the file because the file was zipped 
unzip(zipfile="./data/Dataset.zip",exdir="./data")

path_rf <- file.path("./data" , "UCI HAR Dataset")
files<-list.files(path_rf, recursive=TRUE)
files

## Loading packages needed to perform the task
library(plyr)

## Reading the data as there are different tables provided in the dataset. 

# Renaming as it is difficult to distinguish. 

activitytest  <- read.table(file.path(path_rf, "test" , "Y_test.txt" ),header = FALSE)

activitytrain <- read.table(file.path(path_rf, "train", "Y_train.txt"),header = FALSE)

subjecttrain <- read.table(file.path(path_rf, "train", "subject_train.txt"),header = FALSE)

subjecttest  <- read.table(file.path(path_rf, "test" , "subject_test.txt"),header = FALSE)

featurestest  <- read.table(file.path(path_rf, "test" , "X_test.txt" ),header = FALSE)

featurestrain <- read.table(file.path(path_rf, "train", "X_train.txt"),header = FALSE)

activitylabel <- read.table(file.path(path_rf, "activity_labels.txt"),header = FALSE)

featuresname <- read.table(file.path(path_rf, "features.txt"),head = FALSE)

## Combing the data 1 

# Subject data has combined the previously named subjecttrain data and subjecttest data

subjectdata <- rbind(subjecttrain, subjecttest)

# Activity data has combined the previously named activitytrain data and activitytest data
activitydata <- rbind(activitytrain, activitytest)

# Features data has combined the previously named featurestrain data and featurestest data
featuresdata <- rbind(featurestrain, featurestest)

## Combining previously merged data

names(subjectdata)<-c("subject")

names(activitydata)<- c("activity")

names(featuresdata)<- featuresname$V2

## Final Merged file

# Combinding subjectdata and activity data together

merged <- cbind(subjectdata, activitydata)

# Combinding subjectdata, and activity data that are previously merged with features data. So all three datas are combined. 
finaldata1 <- cbind(featuresdata, merged)

## Renaming

names(finaldata1)<-gsub("^t", "TIME", names(finaldata1))

names(finaldata1)<-gsub("^f", "FREQUENCY", names(finaldata1))

names(finaldata1)<-gsub("Acc", "ACCERLORMETER", names(finaldata1))

names(finaldata1)<-gsub("Gyro", "GYROSCOPE", names(finaldata1))

names(finaldata1)<-gsub("Mag", "MAGNITUDE", names(finaldata1))

names(finaldata1)<-gsub("BodyBody", "BODY", names(finaldata1))

## Creating subsample

# This subsample includes mean and standard deviation only

subsample1 <-featuresname$V2[grep("mean\\(\\)|std\\(\\)", featuresname$V2)]

selected <- c(as.character(subsample1), "subject", "activity" )

## Complete dataset

completedata <- subset(finaldata1,select=selected)

## Final File

finalfinal <- aggregate(. ~subject + activity, finaldata1, mean)

finalfinal <- finalfinal[order(finalfinal$subject,finalfinal$activity),]

## Exporting dataset

write.table(finalfinal, file = "tidydata.txt",row.name=FALSE)
