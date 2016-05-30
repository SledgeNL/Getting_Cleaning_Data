# Getting_Cleaning_Data
Contains material for the final assignment of Getting and Cleaning Data Course

the information from the UCI HAR  (https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip) is manually  downloaded and unzippe, and placed in the working directory of Rstudio
Or rather, the working directory is pointed towards the download folder for this script.

Step 1: import the files that are used in the conversion of the data-layout
Step 2: combine test- and train-files
Step 3: apply sensible variable names
Step 4: combine subject- activity- and measurement files, and transform from a short and wide table to a long and small table. The tiny-data bit, were variables only point to one parameter (i.e. tBodyAccJerkMag-mean() points to 6 parameters (t, Body, Acc, Jerk, Mag, mean())
Step 5: select the right columns, group them so it's possible to average the measurements per group.
Export to .txt-file

Code:
# step 1:
# Assuming the files have been downloaded and unzipped

# set working directory to the folder in which the downloaded zip file is (manual) extracted.

setwd("C:/Users/Henriette Hamer/Documents/A00 GTD/PROJECT BIG DATA/COURSERA/03 Getting and Cleaning Data/week 4/Final Assignment")
path <- getwd()

# files to import in R
# UCI HAR Dataset/activity_labels.txt
# UCI HAR Dataset/features.txt

activity_labels <- read.table(paste0(path, "/", "UCI HAR Dataset/activity_labels.txt"))
features <- read.table(paste0(path, "/", "UCI HAR Dataset/features.txt"))

# UCI HAR Dataset/test/subject_test.txt
# UCI HAR Dataset/test/X_test.txt
# UCI HAR Dataset/test/y_test.txt

subject_test <- read.table(paste0(path, "/", "UCI HAR Dataset/test/subject_test.txt"))
X_test = read.table(paste0(path, "/", "UCI HAR Dataset/test/X_test.txt"))
y_test <- read.table(paste0(path, "/", "UCI HAR Dataset/test/y_test.txt"))

## Inertial Signals files are not used

# UCI HAR Dataset/train/subject_train.txt
# UCI HAR Dataset/train/X_train.txt
# UCI HAR Dataset/train/y_train.txt

subject_train = read.table(paste0(path, "/", "UCI HAR Dataset/train/subject_train.txt"))
X_train = read.table(paste0(path, "/", "UCI HAR Dataset/train/X_train.txt"))
y_train = read.table(paste0(path, "/", "UCI HAR Dataset/train/y_train.txt"))	

## Inertial Signals files are not used


## STEP 2:
# rbind test and train files
subject <- rbind(subject_test, subject_train)
X <- rbind(X_test, X_train)
y <- rbind(y_test, y_train)

## STEP 3:
# Making sensible column names
names(subject)[1] <- "subject"
names(X) <- features$V2

# turning number of activity into something readable, just in case, import dplyr
library(dplyr)
y <- mutate(y, activity = activity_labels$V2[V1])

## select only the columns that have mean() or std() in the name
X_mean_std <- X[, c(grep("mean\\(\\)", names(X)), grep("std\\(\\)", names(X)))]

## STEP 4:
# cbind all of the above

wide_short <- cbind(Subject=subject$subject, Activity=y$activity, X_mean_std) 

# transform to small_long
# as the columns contain multiple measuring parameters, these need to be split - with gather
# the 'separate' step splits the new column across the '-' into 3
# first 'mutate' to get the first character out (Time/Frequency domain)
# second 'mutate' to get the other parameters out. 
# there is no differentiation between 'Body' and 'BodyBody'. 
small_long <- gather(wide_short, var_temp, count, -Subject, -Activity) %>%
        separate(var_temp, c("DomAccInstr", "Var", "Axis")) %>%
        mutate(Domain = as.factor(ifelse(substr(DomAccInstr, 1,1)=="t", "time", "frequency")), 
                AccInstr = as.factor(substr(DomAccInstr, 2, 1000))) %>%
        mutate(Instrument = as.factor(ifelse(grepl("Acc", AccInstr), "accelerometer", ifelse(grepl("Gyro", AccInstr), "gyrometer", NA))),
                Acceleration = as.factor(ifelse(grepl("Body", AccInstr), "body", ifelse(grepl("Gravity", AccInstr),"gravity", NA))),
                Jerk = as.factor(ifelse(grepl("Jerk", AccInstr), "jerk",NA)),
                Magnitude = as.factor(ifelse(grepl("Mag", AccInstr), "mag",NA))) 
                                                      

## STEP 5:
# prepare for mean of groups
# grouped_tidy <- group_by(small_long_temp6, subject, activity)

final_data <- small_long %>% 
        select(c(Subject, Activity, Var, Axis, Domain, Instrument, Acceleration, Jerk, Magnitude,count)) %>%
        group_by(Subject, Activity, Var, Axis, Domain, Instrument, Acceleration, Jerk, Magnitude) %>% 
        summarise_each(funs(mean))

write.table(final_data, "final_data.txt", row.names=FALSE)

