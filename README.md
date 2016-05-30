# Getting_Cleaning_Data
Contains material for the final assignment of Getting and Cleaning Data Course

the information from the UCI HAR  (https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip) is manually  downloaded and unzipped, and placed in the working directory of Rstudio
Or rather, the working directory is pointed towards the download folder for this script.

Step 1: import the files that are used in the conversion of the data-layout

Step 2: combine test- and train-files

Step 3: apply sensible variable names

Step 4: combine subject- activity- and measurement files, and transform from a short and wide table to a long and small table. The tiny-data bit, were variables only point to one parameter (i.e. tBodyAccJerkMag-mean() points to 6 parameters (t, Body, Acc, Jerk, Mag, mean())

Step 5: select the right columns, group them so it's possible to average the measurements per group.

Export to .txt-file

There is extensive commentary in the script file, please read there if you want more information.

