## Load data sources
raw = read.table("UCI HAR Dataset/train/X_train.txt", header = FALSE);
features = read.table("UCI HAR Dataset/features.txt", stringsAsFactors=FALSE, col.names=c("varnum", "feature"));
activities = read.table("UCI HAR Dataset/train/y_train.txt", stringsAsFactors=FALSE, col.names=c("activity"));
activities_labels = read.table("UCI HAR Dataset/activity_labels.txt", stringsAsFactors=FALSE, col.names=c("varnum","feature"));
subjects =  read.table("UCI HAR Dataset/train/subject_train.txt", stringsAsFactors=FALSE, col.names=c("subject"));

# Fix duplicates by giving unique name
dupfix = which(duplicated(features$feature))

for (i in dupfix) {
  
  features$feature[i] = paste(features$feature[i] , "_d" , as.character(i), sep="")
  
}

# Add despriptive labels to activity & add activity to data file
raw$activity = factor(activities$activity, labels=activities_labels$feature);

# Add Subject
raw$subject = subjects$subject;

# Set column names
colnames(raw) = c(features$feature,'activity','subject');



# Repeat the above for test set
raw2 = read.table("UCI HAR Dataset/test/X_test.txt", header = FALSE);
features = read.table("UCI HAR Dataset/features.txt", stringsAsFactors=FALSE, col.names=c("varnum", "feature"));
activities = read.table("UCI HAR Dataset/test/y_test.txt", stringsAsFactors=FALSE, col.names=c("activity"));
activities_labels = read.table("UCI HAR Dataset/activity_labels.txt", stringsAsFactors=FALSE, col.names=c("varnum","feature"));
subjects =  read.table("UCI HAR Dataset/test/subject_test.txt", stringsAsFactors=FALSE, col.names=c("subject"));


dupfix = which(duplicated(features$feature))

for (i in dupfix) {
  
  features$feature[i] = paste(features$feature[i] , "_d" , as.character(i), sep="")
  
}

raw2$activity = factor(activities$activity, labels=activities_labels$feature);
raw2$subject = subjects$subject;
colnames(raw2) = c(features$feature,'activity','subject');

## Combine test & train data
raw3 <- rbind(raw,raw2);

##Select only means and std columns
stdcols <- grep('std()',names(raw));
meancols <- grep('mean()', names(raw));
excld <- grep('meanFreq()', names(raw));
selector <- c(stdcols,meancols,562,563);
selectgo <- selector[-excld];

## Process Data into Tidy Format
# Subset output based on the correct selection of variables
raw_output <- raw3[selectgo];

# Load necessary libraries (assuming packages installed) for casting functions. Using both reshape & reshape2 to be safe.
library(reshape);
library(reshape2);

# Cast variables for each combination of activity & subject
casted_output <- melt(raw_output,id=c("activity","subject"));

# Fetch the means of the variables for each combination of activity and subject
final <- dcast(casted_output, activity + subject ~ variable,mean);

