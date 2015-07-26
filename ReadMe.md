## This is the ReadMe for Coursera Getting and Cleaning Data Week 3 Assignement Submission.

Here I will briefly go over the initial requirements for the assignemnt and then detail my scripts for cleaning the data. 

### Assignemtn Details

For Coursera - Data Science Specialization - Getting and Cleaning Data - Week 3 - Assignment - UCI HAR Dataset

Assignemnt directions direct from the course Assignemnt page:

	*The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis. You will be graded by your peers on a series of yes/no questions related to the project. You will be required to submit: 1) a tidy data set as described below, 2) a link to a Github repository with your script for performing the analysis, and 3) a code book that describes the variables, the data, and any transformations or work that you performed to clean up the data called CodeBook.md. You should also include a README.md in the repo with your scripts. This repo explains how all of the scripts work and how they are connected.  

	*One of the most exciting areas in all of data science right now is wearable computing - see for example this article . Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained: http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones 

	*Here are the data for the project: https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip 

	*You should create one R script called run_analysis.R that does the following:
		*Merges the training and the test sets to create one data set.
		*Extracts only the measurements on the mean and standard deviation for each measurement. 
		*Uses descriptive activity names to name the activities in the data set
		*Appropriately labels the data set with descriptive variable names. 
		*From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

	*Good luck!

### Scripts

I have created 1 script that downloads the UCI HAR Dataset and cleans the data (including procducing outputs) if the data has not already been downloaded and cleaned.

Script for the run_Analysis function (file as "run_analysis.R"):

run_Analysis <- function() {
	
	# Sets up directories for raw data and downloads data if the appropriaqte file paths have not already been created.
		Assignment3Dir <- getwd()

		if(!file.exists("./DATA")) {
			dir.create("./DATA")
		}
		if(!file.exists("./DATA/UCI HAR Dataset")) {
			download.file("http://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip ", destfile = "./DATA/RawData.zip", mode = "wb")
			unzip("./DATA/RawData.zip", exdir = "./DATA")
			file.remove("./DATA/RawData.zip")
		}
	
	# Loads useful libraries.
		library(dplyr)
		library(reshape)
		library(reshape2)
		library(stringr)
		library(devtools)

	# Performs initial clean up of the data to produce full test and train tables.  This includes using descriptive activity names to name the activities in the data set, and appropriately labeling the data set with descriptive variable names. 
		setwd("./DATA/UCI HAR Dataset")
		t <- read.table("activity_labels.txt", sep = "")
		activity_labels <- as.character(t$V2)
		t <- read.table("features.txt", sep = "")
    		features <- t$V2
		
		X_test <- read.table("./test/X_test.txt", sep = "")
    		names(X_test) <- features
    		y_test <- read.table("./test/y_test.txt", sep = "")
    		names(y_test) <- "Activity_Label"
   		y_test$Activity_Label <- as.factor(y_test$Activity_Label)
   		levels(y_test$Activity_Label) <- activity_labels
    		subject_test <- read.table("test/subject_test.txt", sep = "")
    		names(subject_test) <- "subject"
    		subject_test$subject <- as.factor(subject_test$subject)
    		test_bind <- cbind(X_test, subject_test, y_test)
		test_bind$group <- rep("test", times = nrow(test_bind))

		X_train <- read.table("train/X_train.txt", sep = "")
    		names(X_train) <- features
    		y_train <- read.table("train/y_train.txt", sep = "")
    		names(y_train) <- "Activity_Label"
    		y_train$Activity_Label <- as.factor(y_train$Activity_Label)
    		levels(y_train$Activity_Label) <- activity_labels
    		subject_train <- read.table("train/subject_train.txt", sep = "")
    		names(subject_train) <- "subject"
    		subject_train$subject <- as.factor(subject_train$subject)
    		train_bind <- cbind(X_train, subject_train, y_train)
		train_bind$group <- rep("train", times = nrow(train_bind))

	# Combines the test and train data sets and extracts only the the column variables that deal with the mean and standard deviation of the measurements.
		combined <- rbind(test_bind, train_bind)
		i <- 1
		j <- 1
		vec_fil <- vector()
		
		while(i <= 561) {
			if(grepl("mean", names(combined)[i]) | grepl("std", names(combined)[i]) | grepl("Mean", names(combined)[i])) {
				vec_fil[j] <- as.character(names(combined)[i])
				i <- i + 1
				j <- j + 1
			} else {
				i <- i + 1
			}
		}

		combined <- cbind(combined[, names(combined) %in% vec_fil], combined[, c(562, 563, 564)])
		
	# Splits the combined data set into 6 separate data sets, 1 for each of the Activities, and calculates the average of each variable for each subject.
		combined_melt <- melt(combined, id = c("subject", "Activity_Label"), measure.vars = names(select(combined, -(subject:group))))
		melt_WALKING <- filter (combined_melt, Activity_Label == "WALKING")
		melt_WALKING_UPSTAIRS <- filter (combined_melt, Activity_Label == "WALKING_UPSTAIRS") 
		melt_WALKING_DOWNSTAIRS <- filter (combined_melt, Activity_Label == "WALKING_DOWNSTAIRS")
		melt_SITTING <- filter (combined_melt, Activity_Label == "SITTING")
		melt_STANDING <- filter (combined_melt, Activity_Label == "STANDING")
		melt_LAYING <- filter (combined_melt, Activity_Label == "LAYING")
		cast_WALKING <- dcast(melt_WALKING, subject ~ variable,mean)
		cast_WALKING_UPSTAIRS <- dcast(melt_WALKING_UPSTAIRS, subject ~ variable,mean)
		cast_WALKING_DOWNSTAIRS <- dcast(melt_WALKING_DOWNSTAIRS, subject ~ variable,mean)
		cast_SITTING <- dcast(melt_SITTING, subject ~ variable,mean)
		cast_STANDING <- dcast(melt_STANDING, subject ~ variable,mean)
		cast_LAYING <- dcast(melt_LAYING, subject ~ variable,mean)

	# Outputs the tidy data set specified in the assignment.
		setwd(Assignment3Dir)
		if(!file.exists("./tidy_WALKING.txt")) {
			write.table(cast_WALKING, file = "./tidy_WALKING.txt", row.name = FALSE)
		}
		if(!file.exists("./tidy_WALKING_UPSTAIRS.txt")) {
			write.table(cast_WALKING_UPSTAIRS, file = "./tidy_WALKING_UPSTAIRS.txt", row.name = FALSE)
		}
		if(!file.exists("./tidy_WALKING_DOWNSTAIRS.txt")) {
			write.table(cast_WALKING_DOWNSTAIRS, file = "./tidy_WALKING_DOWNSTAIRS.txt", row.name = FALSE)
		}
		if(!file.exists("./tidy_SITTING.txt")) {
			write.table(cast_SITTING, file = "./tidy_SITTING.txt", row.name = FALSE)
		}
		if(!file.exists("./tidy_STANDING.txt")) {
			write.table(cast_STANDING, file = "./tidy_STANDING.txt", row.name = FALSE)
		}
		if(!file.exists("./tidy_LAYING.txt")) {
			write.table(cast_LAYING, file = "./tidy_LAYING.txt", row.name = FALSE)
		}

}