Part 1: Merge the training and the test sets to create one data set
===================================================================

Step 1: Unzip the data in the wd, but leaving the original
subdirectories

    wd<-paste(getwd(),"/UCI HAR Dataset",sep="")

Step 2: Read in all the data

    x.test<-read.table(paste(wd,"/test/x_test.txt",sep=""))
    y.test<-read.table(paste(wd,"/test/y_test.txt",sep=""))
    s.test<-read.table(paste(wd,"/test/subject_test.txt",sep=""))
    x.train<-read.table(paste(wd,"/train/x_train.txt",sep=""))
    y.train<-read.table(paste(wd,"/train/y_train.txt",sep=""))
    s.train<-read.table(paste(wd,"/train/subject_train.txt",sep=""))
    x.labels<-read.table(paste(wd,"/features.txt",sep=""))

Step 3: Merge train and test for s, x, and y

    x.data<-rbind(x.test,x.train)
    y.data<-rbind(y.test,y.train)
    s.data<-rbind(s.test,s.train)

Step 4: Give appropriate labels to variables

    colnames(s.data)<-"Subject"
    colnames(y.data)<-"Activity"
    colnames(x.data)<-x.labels$V2

Step 5: combine s,x,y-datasets into one dataset

    raw<-cbind(s.data,x.data,y.data)

Step 6: Clear intermediate tables to clean up (optional)

    rm(list=c("x.data","y.data","s.data", "x.test","y.test","s.test", "x.train","y.train","s.train","x.labels"))

Part 2: Extract only the measurements on the mean and standard deviation for each measurement
=============================================================================================

Step 1: Filtering out the neccesary columns. We do select
meanFreq-variables as this is a different type of variable-measurement

    raw<-raw[,c(1,563,
           grep("-mean()",colnames(raw)),
           grep("-std()",colnames(raw)))]

Part 3: Use descriptive activity names to name the activities in the data set
=============================================================================

Step 1: Load the activty as a factor

    raw$Activity<-as.factor(raw$Activity)

Step 2: Replace the labels in the data-set

    levels(raw$Activity)<-list(Walking = 1, Walking_Upstairs = 2, Walking_Downstairs=3, Sitting = 4, Standing = 5, Laying = 6)

Part 4: Appropriately label the data set with descriptive variable names
========================================================================

I will not rename any of the original feature-names as they contain
information about how the variabele was measured. If I would rename
them, this information would be lost. Subject and Activity already have
descriptive labels

    raw<-raw

Part 5: From the data set in step 4, create a second, independent tidy data set with the average of each variable for each activity and each subject
====================================================================================================================================================

Step 1: Split the data-frame based on the interaction Subject-Activity

    splits<-interaction(raw$Activity,raw$Subject)
    net<-split(raw,splits, drop=TRUE)

Step 2: Calculate the mean of every column

    net<-sapply(net, function(x) colMeans(x[,-c(1,2)],na.rm=TRUE))

Step 3: Transpose the result

    net<-as.data.frame(t(net))

Step 4: Resplit the interaction

    net$Activity <- sapply(strsplit(as.character(rownames(net)), "\\."), "[", 1)
    net$Subject <- sapply(strsplit(as.character(rownames(net)), "\\."), "[", 2)

Step 5: remove the rownames created by the apply-function

    rownames(net)<-NULL

Step 6: Re-order the columns

    net<-net[,c(81,80,1:79)]

Step 7: Write the table

    write.table(net,"GaCD.txt",row.names = FALSE)
