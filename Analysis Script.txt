## To get the data you download the file and put it in the R Code Lessons folder 
##(in my case this is where i put the data in the R Code Lessons)

<h2>Getting the Data</h2>
# This code will download the file and put the file in the GettingandCleanning folder
if(!file.exists("./GettingandCleaning")){dir.create("./GettingandCleaning")}
fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileUrl,destfile="C:/Users/Admin/Documents/R Code Lessons/Dataset.zip",method="curl")

#This code will unzip the file
unzip(zipfile="C:/Users/Admin/Documents/R Code Lessons/Dataset.zip",exdir="./GettingandCleaning")


#The unzipped files are in the folder UCI HAR Dataset
path_rf <- file.path("./GettingandCleaning" , "UCI HAR Dataset")
files<-list.files(path_rf, recursive=TRUE)
files

<h2>Reading the Data from the files</h2>

#this code will read the Activity files
dataActivityTest  <- read.table(file.path(path_rf, "test" , "Y_test.txt" ),header = FALSE)
dataActivityTrain <- read.table(file.path(path_rf, "train", "Y_train.txt"),header = FALSE)

#this code will read the Subject files
dataSubjectTrain <- read.table(file.path(path_rf, "train", "subject_train.txt"),header = FALSE)
dataSubjectTest  <- read.table(file.path(path_rf, "test" , "subject_test.txt"),header = FALSE)

#this code will read the Feature files
dataFeaturesTest  <- read.table(file.path(path_rf, "test" , "X_test.txt" ),header = FALSE)
dataFeaturesTrain <- read.table(file.path(path_rf, "train", "X_train.txt"),header = FALSE)

<h2>Step 1: Merges the training and the test sets to create one data set</h2>

#This code will link the data tables by rows
dataSubject <- rbind(dataSubjectTrain, dataSubjectTest)
dataActivity<- rbind(dataActivityTrain, dataActivityTest)
dataFeatures<- rbind(dataFeaturesTrain, dataFeaturesTest)

# This will set the names to variables
names(dataSubject)<-c("subject")
names(dataActivity)<- c("activity")
dataFeaturesNames <- read.table(file.path(path_rf, "features.txt"),head=FALSE)
names(dataFeatures)<- dataFeaturesNames$V2

#This will merge the columns to get the data frame data for all data
dataCombine <- cbind(dataSubject, dataActivity)
Data <- cbind(dataFeatures, dataCombine)

<h2> Step 2:Extracts only the measurements on the mean and standard deviation for each measurement</h2>

#This will Subset the Name of Features by measurements on the mean and standard deviation
subdataFeaturesNames<-dataFeaturesNames$V2[grep("mean\\(\\)|std\\(\\)", dataFeaturesNames$V2)]

#This will Subset the data frame data by seleted names of Features
selectedNames<-c(as.character(subdataFeaturesNames), "subject", "activity" )
Data<-subset(Data,select=selectedNames)

#This will allow you to check the structures of the data frame data
str(Data)

<h2>Step 3:Uses descriptive activity names to name the activities in the data set</h2>

#This will read descriptive activity names from activity_labels.txt
activityLabels <- read.table(file.path(path_rf, "activity_labels.txt"),header = FALSE)
head(Data$activity,30)

<h2>Step 4:Appropriately labels the data set with descriptive variable names</h2> 

#As you can see from this code the variables activity,subject and names of the activities 
#have been labelled using descriptive names.Feteatures will labelled using descriptive variable names;
#.prefix t is replaced by time
#.Acc is replaced by Accelerometer
#.Gyro is replaced by Gyroscope
#.prefix f is replaced by frequency
#.Mag is replaced by Magnitude
#.BodyBody is replaced by Body

names(Data)<-gsub("^t", "time", names(Data))
names(Data)<-gsub("^f", "frequency", names(Data))
names(Data)<-gsub("Acc", "Accelerometer", names(Data))
names(Data)<-gsub("Gyro", "Gyroscope", names(Data))
names(Data)<-gsub("Mag", "Magnitude", names(Data))
names(Data)<-gsub("BodyBody", "Body", names(Data))
names(Data)


<h2>Step 5:Creates a second, independent tidy data set with the average of each variable for each activity and each subject</h2>

#This code will allow you to create an independent tidy data set, this will be created with the average of each variable for each activity #and each subject based on the data set in step 4.

library(plyr)
Data2<-aggregate(. ~subject + activity, Data, mean)
Data2<-Data2[order(Data2$subject,Data2$activity),]
write.table(Data2, file = "tidydata.txt",row.name=FALSE)

