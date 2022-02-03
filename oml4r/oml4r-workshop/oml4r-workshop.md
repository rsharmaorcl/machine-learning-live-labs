## About this workshop

Oracle Machine Learning for R (OML4R) enables you to use R (a leading statistical programming language) for statistical analysis, data exploration, machine learning, and graphical analysis of data stored in an Oracle database. Using OML4R allows you to benefit from the simplicity of R and the power of Oracle Database without the need to deal with the complexities of sourcing, moving, and securing data. OML4R was formerly known as Oracle R Enterprise (ORE).

In this introductory workshop, you will use a dataset representing about 15,000 customers of an insurance company. This dataset is contained in a database table, CUST_INSUR_LTV. Each customer record in the dataset has 31 attributes. Your goal is to train machine learning models to predict a given customer's lifetime value (or LTV) and a four-level category representation of their lifetime value or LTV (which is represented by an LTV_BIN assignment of each customer record). You will use a regression algorithm to predict LTV, and a classification algorithm to predict customers’ LTV_BIN assignments (LOW, MEDIUM, HIGH, or VERY HIGH)

Note: In marketing, the lifetime value (LTV) of a customer is an estimate of the net profit attributed to a given customer relationship over the customer lifetime.

### Estimated Lab Time: 2 Hours

### Objectives

In this lab, you will:

* Establish a connection from RStudio Web to your Oracle Database instance.
* Explore, visualize, and prepare data for analysis and machine learning.
* Use R for exploratory data analysis, data visualization, data organization (e.g., splitting data in train and test sets),
* Use Attribute Importance and Principal Component Analysis
* Use an OML4R in-database regression algorithm for building a machine learning model for estimating customer life-time value (LTV)
* Use an OML4R in-database classification algorithm for building a machine learning model for predicting LTV_BIN
* Assess model quality using RMSE (Root Mean Squared Error) for regression and a confusion matrix for classification

### Note:

1. AutoML is not available for OML4R. 
2. AutoML is a feature of OML4Py and is available with Autonomous Database through the OML AutoML UI.
3. We will not be using Autonomous Database as OML4R is currently not available for Autonomous Database.

### Pre-requisites

* Oracle Database 21c, 19c, or 18c installed on-premises (or in a VM in Cloud)
* R, RStudio Server, and required libraries

## Task 1: Connect to RStudio Client and Establish Database Connection (DONE)

RStudio provides an IDE (Integrated Development Environment) for R with a GUI (Graphical User Interface) and BUI (Browser User Interface). Note that RStudio Server has been pre-configured on your VM that is running the database server.

You will be running all the lab steps in the RStudio R Script window as shown below.

1.1: Point browser to RStudio Web

```

http://<ip-address>:8787

```

Alternatively, you can use RStudio Desktop as your IDE, if you prefer.

1.2: Provide Username & Password

Use the provided username and password to connect to the database schema.

```

Username: omluser 
Password: MLlearnPTS#21_

```
            
![connect](./images/connect-1.png)

The RStudio interface looks as follows. Notice the multiple windows marked for their function.

![rstudio](./images/rstudio-1.png)

The RStudio Graphical User Interface (GUI) provides four different panels: Console, Scripts, Environments, and Plots. Instead of working ad-hoc, writing scripts allows you to make your code and workflow reproducible. 

1.3: Install packages

```

if (!require(“ORE”)) install.packages(“ORE”) 
if (!require(“dplyr”)) install.packages(“dplyr”) 
if (!require(“OREdplyr”)) install.packages(“OREdplyr”) 

```

The above checks the existence of the given package in the present installation. It only installs the package if the given package is not already installed, thus eliminating redundant installation of packages.

1.4: Load Libraries

The library() function call installs attaches the given library in memory and makes the included functions available. If a call to a function is made and the corresponding library is not already attached, you get an error.

```

library(ORE)
library(dplyr)
library(OREdplyr)

```

ORE - The ORE libraries are a set of packages that contains many useful R functions.

dplyr - The standard dplyr R package provides grammar of data manipulation, which gives a consistent set of verbs that help you solve the most common data manipulation challenges

OREdplyr - The OREdplyr package is an overloaded package that provides much of the dplyr functionality. It extends the ORE transparency layer for in-database execution of dplyr function calls. OREdplyr allows users to avoid costly movement of data while scaling to larger data volumes. Further, using the transparency layer operations are not constrained by R client memory, the latency of data movement, or single-threaded execution. 

1.5: Set Global Options to Disable Unnecessary Warning Messages

Turn off warnings to simplify the flow.

```

options(ore.warn.order=FALSE)

```

1.6: Connect to the Database

An Oracle 21c database instance (named MLPDB1) has been provisioned for you to run this lab. Connect to the provided database using the ore.connect() function as follows:

```

ore.connect(user="oml_user",
            conn_string="MLPDB1",
            host=<hostname>,
            password=<password>,
            all=TRUE)

```

Your database connection is to the database schema where the data resides. The connection port defaults to 1521. 

By specifying “all = TRUE” in the connection specifications, proxy objects are automatically loaded for all tables in the target schema to which you are connecting.

Example:

```

ore.connect(user="oml_user",
            conn_string="MLPDB1",
            host="130.61.44.205",
            password="MLlearnPTS#21_",
            all=TRUE)

```

1.7: Check Database Connection

```

ore.is.connected()

```

If needed, you can use ore.disconnect() call to explicitly disconnect the database session. As we will discuss later, if you do this, then any temporary table, view, or model proxy objects will be automatically deleted.

1.8: List Database Objects

The ore.ls() function returns all OML4R proxy objects. Use the ore.ls() function call to list tables in the database schema you are connected to, which appear as ore frames in your OML4R session. The conversion is transparent and it is enabled by the OML4R transparency layer.

```

ore.ls()

```

![ls](./images/ls-1.png)


## Task 2: Explore Data Using R - Statistical Functions (DONE)

Exploratory Data Analysis is the process of visualizing and analyzing data to develop better understanding of the data and gain insight into the data.


1: Check ‘class’ of Object

```

class(CUST_INSUR_LTV)

```

The database table appears as an "ore.frame". An ore.frame is a proxy object - the R object representation - of the CUST_INSUR_LTV table in the database. 

There are 6 types of objects in R Programming. They include vector, list, matrix, array, factor, and data frame. An ore.frame object represents a relational (SQL) query to an Oracle Database instance. The class ore.frame inherits from data.frame and overloads many data.frame functions.

2: Get List of Column Names (‘colnames’)

```

colnames(CUST_INSUR_LTV)

```

The column names appear in an ordered list and can be referenced based on this order.


3: Check Object or Frame Dimensions

```

dim(CUST_INSUR_LTV)

```

The dimensions represents the number of rows (i.e., records or observations) and number of columns (or variable, or attributes) in the given frame (table).

Your result should be: 15342 31, which means there are 15342 records or observations (rows) in the table and each record has 31 attributes (columns).

            
![basic](./images/basics-1.png)            
            
4: Summary (Categorical and Numerical Variables)

The summary() function is a built-in R function that can help you get a quick survey of the distribution of your variables in the dataset. The output returns descriptive statistics such as the minimum, the 1st quantile, the median, the mean, the 3rd quantile, and the maximum value of our input data.

```

summary(CUST_INSUR_LTV[,1:31])

```

You can also specify one, multiple, or a range of columns to see the summary statistics of those columns.

![summary](./images/summary-1.png)

The summary() call summarizes all the specified attributes, along with statistics for frequency counts, minimum values, maximum values, etc. Notice the reference to the given set of columns (attributes) of the table using the order numbers for the columns. You can also see a summary of individual attributes by specifying a specific column by name or number.

5: Statistical Functions (min, max, mean, mode, median, range, unique, quantile)

You can also use simple statistical functions on your dataset or data elements as desired. For example,

```

min(CUST_INSUR_LTV$SALARY)

max(CUST_INSUR_LTV$AGE)

```

Note: Mean is the statistical average.

```

mean(CUST_INSUR_LTV$SALARY)

mean(CUST_INSUR_LTV$LTV)

```

Note: Mode is the most frequently occurring observation or attribute.

```

names(table(CUST_INSUR_LTV$N_OF_DEPENDENTS))[table(CUST_INSUR_LTV$N_OF_DEPENDENTS)==max(table(CUST_INSUR_LTV$N_OF_DEPENDENTS))]

```

Note: Median represents the middle value.

```

median(CUST_INSUR_LTV$AGE)

```

Range provides end-to-end range of numeric values.

```

range(CUST_INSUR_LTV$SALARY)

```

Unique provides the set of unique or distinct values in the data set for a given attribute or variable.

```

unique(CUST_INSUR_LTV$REGION)

```

In some cases, quantiles may be useful to identify outlier limits. The generic function 'quantile()' produces sample quantiles corresponding to the specified probabilities. The smallest observation corresponds to a probability of 0 and the largest to a probability of 1.

```

quantile(CUST_INSUR_LTV$SALARY, 0.025)

quantile(CUST_INSUR_LTV$SALARY, 0.975)

```

Your results should show the following outputs.

![stats](./images/stats-1.png)


## Task 3: Explore Data Using R - Data Visualization (DONE)

Exploratory Data Analysis includes the process of visualizing data for better understanding and insight.

1: Simple Plot

Draw a simple and quick plot of customer salary.

This basic plot illustrates the overall distribution of salary and the range within which it falls. You clearly see a dense band where most customers fall between about $50K to about $80K.

```

plot(CUST_INSUR_LTV$SALARY)
plot(CUST_INSUR_LTV$SALARY/1000, xlab = "Customer", ylab = "Salary in K$", col = "darkred", main = "Customer Salary Plot")

```

Your plot should look as follows.

![plot](./images/plot-1.png)

2: Boxplot 

Plot the age attribute using a simple boxplot.

A simple boxplot can help you quickly see the concentration of customers in specific bands.

```

out <- boxplot.stats(CUST_INSUR_LTV$AGE)$out
boxplot(CUST_INSUR_LTV$AGE, xlab = "Boxplot (AGE)", col = "darkred", horizontal=TRUE)
text(x=fivenum(x), labels = fivenum(x), y=1.35)
mtext(paste("Outliers: ", paste(unique(out), collapse = ", ")))

```

Your output should look as follows.

![boxplot](./images/boxplot-1.png)

A boxplot displays distribution of data based on a 5-number summary (“minimum”, first quartile (Q1), median, third quartile (Q3), and “maximum”). A boxplot makes it easy to see outliers and what their values are. It can also indicate if your data is symmetrical, general grouping of data, and data skew.

The above boxplot illustrates the distribution of data with a smallest (not minimum) value of 0, a highest (not maximum) value of 74, and an interquartile range from 27 to 46, with a median right in the center of these two (at 36) as depicted by the solid line. Values ranging from 75 to 84 are shown as outliers.

Did you notice the parallel between the summary() function output and the box plot?

3: Histogram

Plot a histogram for customer salary data.

Histograms help see distribution of data in range bands. Note, the hist() function uses the Sturges method by default to identify appropriate breaks.

```

hist(CUST_INSUR_LTV$SALARY/1000,
main="Customer Salary Data",
xlab="Salary($K)",
xlim=c(20,100),
col="darkred”,
freq=TRUE)

```

Your output should look as follows.
            
![histogram](./images/histogram-1.png)

4: Pie Chart

Generate a pie chart for distribution of customers by region.

```

pie(table(CUST_INSUR_LTV$REGION), main = "Customer Distribution by Region", clockwise = TRUE)  

```

Your output should look as follows.
            
            
![pie](./images/pie-1.png)

Note that CLOCKWISE signifies to use alphabetical order. 


5: ggplot2

Use the package ggplot2 to generate a plot for visualizing LTV distribution for various regions.

When using third-party packages, the data needs to be loaded into R memory, from the database. For this we use the ore.pull() function. Note that CUST_INSUR_LTV is an ore.frame and once the data is pulled, it is an R data.frame. Users must take into account the size of a table before attempting to load it into memory. 

```

CIL <- ore.pull(CUST_INSUR_LTV)
row.names(CIL) <- CIL$CUST_ID
CIL %>% ggplot(aes(x=REGION,y=LTV,color=REGION)) + geom_boxplot() 

```

Your output should look as follows.

![ggplot](./images/ggplot-1.png)

Let’s look at another plot using ggplot2 for marital status.

```

CIL %>% ggplot(aes(x=MARITAL_STATUS)) + geom_histogram(stat="count", color = "darkred", fill = "#69b3a2")  

```

Your output should look as follows.

![ggplot](./images/ggplot-2.png)

Let’s also look at the age distribution in the customer base.

```

print(CIL %>% ggplot(aes(x=AGE)) + geom_density(stat="count", , color = "darkred", fill = "#69b3a2"))

```

Your output should look as follows.

![ggplot](./images/ggplot-3.png)


## Task 4: Explore Data Using R - Targeted Data Exploration (DONE)

Exploratory Data Analysis may also include the process of ad-hoc data analysis, typically based on intuition or existing insights.

1: Filtering Data and Aggregate Data View

Use the filter() function to filter data based on given criteria.

```

nrow(CUST_INSUR_LTV)

nrow(filter(CUST_INSUR_LTV, REGION == "NorthEast"))

nrow(filter(CUST_INSUR_LTV, SALARY > 110000))

nrow(CUST_INSUR_LTV %>% filter(SALARY > mean(SALARY, na.rm = TRUE)))

```

Note that the 'na.rm' specification directs treatment of missing values. 'na.rm = TRUE' removes missing values from the dataset if they are coded as NA.

Use the aggregate() function to group data based on given column or set of columns. Let us look at the data from different angles.

```

remove(CUST_INSUR_LTV)

aggregate(CUST_INSUR_LTV$LTV_BIN, by = list(LTV_BIN = CUST_INSUR_LTV$LTV_BIN),FUN = length)

aggregate(CUST_INSUR_LTV$REGION, by = list(REGION = CUST_INSUR_LTV$REGION),FUN = length)

aggregate(CUST_INSUR_LTV$MARITAL_STATUS, by = list(MARITAL_STATUS = CUST_INSUR_LTV$MARITAL_STATUS),FUN = length)

aggregate(CUST_INSUR_LTV$SEX, by = list(SEX = CUST_INSUR_LTV$SEX),FUN = length)

aggregate(CUST_INSUR_LTV$N_MORTGAGES, by = list(N_MORTGAGES = CUST_INSUR_LTV$N_MORTGAGES),FUN = length)

aggregate(CUST_INSUR_LTV$N_MORTGAGES, by = list(N_MORTGAGES = CUST_INSUR_LTV$N_MORTGAGES),FUN = length)

aggregate(CUST_INSUR_LTV$HAS_CHILDREN, by = list(HAS_CHILDREN = CUST_INSUR_LTV$HAS_CHILDREN),FUN = length)

aggregate(CUST_INSUR_LTV$HOUSE_OWNERSHIP, by = list(HOUSE_OWNERSHIP = CUST_INSUR_LTV$HOUSE_OWNERSHIP), FUN = length)

aggregate(CUST_INSUR_LTV$BUY_INSURANCE, by = list(BUY_INSURANCE = CUST_INSUR_LTV$BUY_INSURANCE), FUN = length)


```

You should see output as follows.

![aggregate](./images/aggregate-1.png)

You can slice and dice data as you wish. For example, let’s say you wish you understand the distribution of male customers in the northeast region who do or do not currently buy insurance.

```

remove(CUST_INSUR_LTV)

row.names(CUST_INSUR_LTV) <- CUST_INSUR_LTV$CUST_ID

CUST_INSUR_LTV <- (CUST_INSUR_LTV %>% filter(REGION == "NorthEast", na.rm = TRUE) %>% filter(SEX == "M"))

aggregate(CUST_INSUR_LTV$BUY_INSURANCE, by = list(BUY_INSURANCE = CUST_INSUR_LTV$BUY_INSURANCE),FUN = length)

```

2: Missing and Zero Values

Check the number of observations (records) where customer age is specified less than 1.

```

nrow(CUST_INSUR_LTV %>% filter(AGE < 1, na.rm = TRUE))

```

3: Removing & Adding Variables (Columns)

```

dim(CUST_INSUR_LTV)
CUST_INSUR_LTV <- subset(CUST_INSUR_LTV, select = -c(LTV_BIN))
dim(CUST_INSUR_LTV)
CUST_INSUR_LTV$LTV_BIN_PRED <- ""
dim(CUST_INSUR_LTV)
colnames(CUST_INSUR_LTV)
remove(CUST_INSUR_LTV)
colnames(CUST_INSUR_LTV)

```
            
You should see output as follows.

![rmadd](./images/rmadd-1.png)

4 Fine tuning the given dataset

Check for the existence of duplicate data in the dataset.

```

CXL <- ore.pull(CUST_INSUR_LTV)
class(CXL)
nrow(CXL %>% get_dupes(CUST_ID))
remove(CXL)

```

Remove data from the frame where customer age is zero or not specified.

```

dim(CUST_INSUR_LTV)
CUST_INSUR_LTV <- CUST_INSUR_LTV %>% filter(AGE > 1)
dim(CUST_INSUR_LTV)
remove(CUST_INSUR_LTV)

```

You should see output as follows.

![filter](./images/filter-1.png) <— REDO


5: Assess Attribute Importance For LTV_BIN Prediction

Use Attribute Importance to identify attributes of high relevance in predicting target dependent variable.

The ore.odmAI() function can be used to run Attribute Importance on the given dataset. This helps identify attributes in the given dataset that are important in predicting the given dependent attribute (LTV_BIN in this case). 

In order to use the ore.odmAI() function effectively, you should first exclude the most significant dependent attributes from the data frame. For example, exclude LTV attribute when predicting LTV_BIN and exclude LTV_BIN when predicting LTV, since LTV_BIN was derived from LTV.

```

CIL <- CUST_INSUR_LTV
dim(CIL)
CIL$LTV <- NULL
dim(CIL)
class(CIL)

```

Your output should look as follows.

![ai](./images/ai-1.png)

You could also exclude a column using the R Formula itself. For example use (LTV_BIN ~ . - CUST_ID) to remove the CUST_ID column from the dataset.

Notice the dimensions now show (15342 30), and not (15342 31) as was originally the case. Now, run the ore.odmAI() function to identify ordered importance of attribute for target variable LTV_BIN. Note that CIL is an ore.frame object.

```

ltvbinai <- ore.odmAI(LTV_BIN ~ ., CIL)
class(ltvbinai)
print(ltvbinai)

```

Assess the output as it lists all the important attributes and their relative influence on the target attribute (LTV_BIN).

Your result should look like the ranking below with House Ownership as the highest importance for determine LTV_BIN, and Customer ID as the least important.

Your output should look as follows.

![ai](./images/ai-2.png)

Save the generated model in datastore.

```

ore.save(ltvbinai, name = 'ltvbinai’, overwrite = TRUE)
ore.datastore()

```

Notice that if the datastore already exists by a given name, an error is flagged. You may overwrite the object, if appropriate, or save it with a different (unique) name.

Your output should look as follows.

#![ai](./images/ai-3.png)


6: Assess Attribute Importance For LTV Prediction

Use Attribute Importance (the ore.odmAI() function) to identify attributes of relative importance for a LTV prediction with the given dataset.

```

CIL <- CUST_INSUR_LTV
CIL$LTV_BIN <- NULL
dim(CIL)

ltvai <- ore.odmAI(LTV ~ ., CIL)

class(ltvai)  
print(ltvai)

ore.save(ltvai, name = 'ltvai', overwrite = TRUE)
ore.datastore()

```

Attribute importance ranks attributes according to their significance in predicting a target. The ore.odmAI() function produces a ranking of attributes and their importance expressed as a fraction.

As seen below, Attribute Importance (AI) ranks attributes according to their significance in predicting a target. 

![ai](./images/ai-4.png)

Do you notice the difference between AI output for LTV and for LTV_BIN?.

7: Principal Component Analysis

Principal Component Analysis (PCA) is a technique used for exploratory data analysis, and to visualize the existing variation in a dataset that has several variables. PCA is particularly helpful when dealing with wide datasets (when each record has many attributes). PCA allows you to simplify a dataset by turning the original (many) variables into a smaller number of what are termed as "Principal Components".

```

prc0 <- prcomp(~  HOUSE_OWNERSHIP + N_MORTGAGES + MORTGAGE_AMOUNT + AGE + SALARY + N_OF_DEPENDENTS, data = CUST_INSUR_LTV, scale = TRUE)

summary(prc0)

```

Note: The prcomp() function is overloaded to build a model on an ore.frame. Setting the argument scale=TRUE, standardizes the input data so that it has zero mean and variance one before doing PCA.

The output below should illustrate the relative significance of the principal components used from the dataset.

![pca](./images/pca-1.png)

The summary() function in the result object shows standard deviation, proportion of variance explained by each principal component, and the cumulative proportion of variance.

8 Save the object in a datastore

```

ore.save(prc0, name = "PRC0”, overwrite = TRUE)
ore.datastore()
ore.datastore(name = "PRC0”, type = c("all"))

```

Your output should look as follows.

![pca](./images/pca-2.png)


If you do not want to overwrite but append the object to the datastore, you can use "append = TRUE".


## Task 5: Build Regression Model for LTV Prediction and Evaluate Model (DONE)

In this task, we are going to 

1: Create Ordered ORE Frame

Most operations in R do not need an ordered frame. Therefore, usually use.keys is set to FALSE. However, for sampling and partitioning etc. ordering of frames is necessary. Since you will be sampling data at this stage, use the primary key of a database table to order an ore.frame object. 

The set.seed() function sets the seed of R‘s random number generator, which is useful for creating simulations or random objects that can be reproduced.

This example ensures that CUST_INSUR_LTV is an ordered ore.frame by assigning the values of the CUST_ID column as the row names of CUST_INSUR_LTV.

```

set.seed(1)

CIL <- CUST_INSUR_LTV
row.names(CIL) <- CIL$CUST_ID

head(row.names(CIL))

```

The output should show the following.

![rownames](./images/rownames-1.png)

Note: Using an ordered ore.frame proxy object can result in certain overheads for a large data set. This is because ordering (sorting) of the result set is performed, which can be time consuming. Therefore, although OML4R attempts to create ordered ore.frame objects by default if a primary key is specified, unordered frames can be made ordered and vice-versa by using the row.names() function.

2: Partition Data for Training & Testing

Split the dataset into two buckets - training data set (~70%), and testing data set (~30%).

The 'seed' can be set to any value. Setting a 'seed' ensures the same output is reproduced by the R psuedonumber generator, when you rerun your code. This can be helpful for consistency and debugging purposes.

```

set.seed(1)

sampleSize <- 4500

ind <- sample(1:nrow(CIL),sampleSize)
group <- as.integer(1:nrow(CIL) %in% ind)
CIL.train <- CIL[group==FALSE,]
CIL.test <- CIL[group==TRUE,]

```

Note that the sample size is specified as 4500 (about 30% of the dataset volume).

3: Check Class and Dimensions of Training and Test Datasets

```

class(CIL.train)
dim(CIL.train)
class(CIL.test)
dim(CIL.test)

```

Your output should look as follows.

![dimtraintest](./images/dimtraintest-1.png)

4: Build Model

Build a REGRESSION MODEL to predict customer LTV using the training data set.

```

ore.fit.glm <- ore.odmGLM(LTV ~ HOUSE_OWNERSHIP + N_MORTGAGES + MORTGAGE_AMOUNT + N_TRANS_WEB_BANK + N_OF_DEPENDENTS, data = CIL.train, ridge=TRUE)

ore.fit.glm %>% print()

```

Your output should look as follows.

![orefitglm](./images/orefitglm1-1.png)

5: Check Model Details

Check details of the REGRESSION MODEL created above.

```

class(ore.fit.glm)

summary(ore.fit.glm)

ore.fit.glm$formula

head(ore.fit.glm$residuals)

```

Your output should look as follows.

![orefitglm](./images/orefitglm-2.png)

6: Generate Predictions

Generate LTV predictions using ore.predict.

The ore.predict function is invoked on a model. For example, the following code generates predictions (ore.fit.glm.pred) by invoking ore.predict() on the ore.fit.glm model produced above and uses CIL.test dataset to score the model.

```

ore.fit.glm.pred = ore.predict(ore.fit.glm, newdata = CIL.test)

head(ore.fit.glm.pred)

```

Note: Using the ore.predict function to score data in the database allows the use of R-generated models to score in-database data (in an ore.frame object). The ore.predict function maximizes the use of Oracle Database as a compute engine which provides a commercial grade, high performance, scalable scoring engine.

Your output should look as follows.

![orefit1glm](./images/orefitglm-3.png)


7: Compare Actual and Predicted Values

```

ore.fit.glm.res <- ore.predict(ore.fit.glm, CIL.test, se.fit = TRUE, interval = "prediction")
CIL <- cbind(CIL, ore.fit.glm.res)
head(select (CIL, LTV, PREDICTION))

```

Your output should look as follows.

![orefitglm](./images/orefitglm-4.png)


8: Compute RMSE

Check Root Mean Squared Error (RMSE) to assess prediction accuracy as produced by the model.

Root Mean Square Error (RMSE) is a metric that indicates the average distance between the predicted values from a model and the actual values in the dataset. A lower RMSE is desirable. Lower RMSE indicates a better “fit” of the model for the given dataset.

```

ans <- ore.predict(ore.fit.glm, newdata = CIL.test, supplemental.cols = 'LTV')
localPredictions <- ore.pull(ans)
ore.rmse <- function (pred, obs) {
sqrt(mean(pred-obs)^2)
}

ore.rmse(localPredictions$PREDICTION, localPredictions$LTV)

```

Your output should show the following:

![orefitglm](./images/orefitglm-5.png)

Recall that the average (mean) LTV as calculated previously was $22266.67. Thus a value of 48.46 is relatively low, suggesting a good model fit.

Plot the predictions alongside actual LTV value.

```

localPredictions %>% ggplot(aes(x=PREDICTION, y=LTV, color=PREDICTION)) + geom_point()

```

Your output should show the following:

![orefitglm](./images/orefitglm-6.png)


RMSE is a useful way to assess the quality of a regression model. When comparing two regression models, a larger RMSE indicates a larger gap between the predicted and observed values, which means a poorer model fit. In the same way, a smaller RMSE that indicates a better model.  However, there is no ideal value for RMSE as it depends on the magnitude of the measure.

9: Store (Save) Model Proxy Object in Database

```

ore.save(ore.fit.glm, name = "ore.fit.glm", overwrite = TRUE)
ore.datastore()

```

Note that in-database models already exist in the database. What we are doing here is to save the model proxy object so the model is not automatically deleted when you close the database connection. This is missing the datastore name in the arguments.  Explain more what is going on here. 

## Task 6: Build Classification Model for LTV_BIN Prediction  and Evaluate Model

1: Create Ordered ORE Frame

Use the primary key of a database table to order an ore.frame object.

The set.seed() function sets the seed of R‘s random number generator, which is useful for creating simulations or random objects that can be reproduced.

This example ensures that CUST_INSUR_LTV is an ordered ore.frame by assigning the values of the CUST_ID column as the row names of CUST_INSUR_LTV.

```

set.seed(1)
CIL <- CUST_INSUR_LTV
row.names(CIL) <- CIL$CUST_ID
head(row.names(CIL))

```

Note: Using an ordered ore.frame object that is a proxy for a SQL query can be time-consuming for a large data set. Therefore, although OML4R attempts to create ordered ore.frame objects by default, it also provides the means of creating an unordered ore.frame object.

2: Partition Data For Training & Testing

Partition the original dataset for training and testing purposes.

Split the dataset into two buckets - training data set (~70%), and testing data set (~30%).

The 'seed' can be set to any value. Setting a 'seed' ensures the same output is reproduced by the R psuedo-number generator, if you run the code again. This can be helpful for consistency and debugging purposes.

```

set.seed(1)
sampleSize <- 4500
ind <- sample(1:nrow(CIL),sampleSize)
group <- as.integer(1:nrow(CIL) %in% ind)
CIL.train <- CIL[group==FALSE,]
CIL.test <- CIL[group==TRUE,]

```


3: Check Class and Dimensions of Training and Test Datasets

```

class(CIL.train)
dim(CIL.train)
class(CIL.test)
dim(CIL.test)

```

Your results should be as follows.

![dimtraintest-2](./images/dimtraintest-2.png)


4: Build Model

Build a CLASSIFICATION MODEL using NAIVE BAYES algorithm and the training data set for predicting customer LTV_BIN assignment.

```

ore.fit.nb <- ore.odmNB(LTV_BIN ~ HOUSE_OWNERSHIP + N_MORTGAGES + MORTGAGE_AMOUNT + N_TRANS_WEB_BANK + N_OF_DEPENDENTS, data = CIL.train)

ore.fit.nb %>% print()

```

Your results should be as follows.

![orefitnb](./images/orefitnb-1.png)


5: Check Model Details

```

summary(ore.fit.nb)

names(ore.fit.nb)

ore.fit.nb$formula

```

Your results should be as follows.

![orefitnb](./images/orefitnb-1a.png)
![orefitnb](./images/orefitnb-1b.png)

6: Generate Predictions for LTV_BIN

```

ore.fit.nb.res <- ore.predict(ore.fit.nb, newdata = CIL.test)

head(ore.fit.nb.res,10)

```

Your results should be as follows.

![orefitnb](./images/orefitnb-2.png)


7: Compute Confusion Matrix

Now, let us look at the confusion matrix to compare the aggregate predictions with actual data.

```

with(ore.fit.nb.res, table(LTV_BIN,PREDICTION, dnn = c("Actual","Predicted")))

```

Your results should be as follows.

![orefitnb](./images/orefitnb-3.png)


A confusion matrix is used to describe the performance of a classification model on a test dataset for which the actual or true values are known. It is usually presented in a table format. The confusion matrix depicts TRUE POSITIVES, TRUE NEGATIVES, FALSE POSITIVES, and FALSE NEGATIVES. The accuracy is easy to calculate by computing (TP + TN / TOTAL SAMPLE SIZE).

8: Store (Save) Model In Database

```

ore.save(ore.fit.nb, name = "ore.fit.nb”, overwrite = TRUE)
ore.datastore()

```

9: Optionally, use another algorithm to create a different model and compare the results. 

For example, the following creates a classification model using the DECISION TREE algorithm. 

Remove the LTV_BIN column.

```

CIL <- CUST_INSUR_LTV
CIL$LTV_BIN <- NULL
dim(CIL)

```

Fit the model with training data.

```

ore.fit.dt <- ore.odmDT(LTV_BIN ~ ., data = CIL.train)
ore.fit.dt <- ore.odmDT(LTV_BIN ~ HOUSE_OWNERSHIP + N_MORTGAGES + MORTGAGE_AMOUNT + N_TRANS_WEB_BANK + N_OF_DEPENDENTS, data = CIL.train)
ore.fit.dt %>% print()
summary(ore.fit.dt)
names(ore.fit.dt)
ore.fit.dt$formula

```

Generate the confusion matrix.

```

ore.fit.dt.res <- predict (ore.fit.dt, CIL.test, "LTV_BIN")
head(ore.fit.dt.res,10)
with(ore.fit.dt.res, table(LTV_BIN,PREDICTION, dnn = c("Actual","Predicted")))

```

Your results should be as follows.

![orefitnb](./images/orefitnb-4.png)


You may save this model in the datastore as follows:

```

ore.save(ore.fit.dt, name = "ore.fit.dt”, overwrite = TRUE)
ore.datastore()

```

How does the confusion matrix compare with the one generated previously?


## Task 7: Use Embedded R Functions to Leverage In-Database Parallel Processing (DONE)

Some of the most significant benefits of using OML4R can be derived from using Embedded R execution in your applications. Embedded R execution allows you to store and run R scripts in the database using R and SQL interfaces.

1. Import libraries and connect to the database

```

library(ORE)
options(ore.warn.order=FALSE)

ore.connect(user="oml_user",
            conn_string="MLPDB1",
            host="rinst5d",
            password="oml_user",
            all=TRUE)

ore.is.connected()

```

Your output should look as follows:

![embr](./images/embr-1.png)

2. Select Algorithm and Build Machine Learning Model  

Let us first invoke a script with table as input and test using open source R test and the local R data frame.

```

cust_insur_ltv_loc <- ore.pull(CUST_INSUR_LTV)
class(cust_insur_ltv_loc)

```

```

glm.fit <- function(dat){
             glm(LTV ~ N_MORTGAGES + MORTGAGE_AMOUNT + N_OF_DEPENDENTS, data = dat)}

mod.loc <- glm.fit(cust_insur_ltv_loc)
mod.loc
class(mod.loc)

```

Your output should look as follows:

![embr](./images/embr-2.png)


Now, let us use ore.TableApply function with a database proxy table as input. This allows for in database processing and eliminates the need to “pull” data out.

```

glm.fit.oml4r <- ore.tableApply(CUST_INSUR_LTV, FUN=glm.fit)
glm.fit.oml4r
class(glm.fit.oml4r)
ore.pull(glm.fit.oml4r)

```

Your output should look as follows:

![embr](./images/embr-3.png)


3. Save Model in Datastore and 

Save model in data store.

```

ore.save(glm.fit.oml4r, name = “GLMFITOML4R”)
ore.datastore()

```

Create function for generating predictions for a given dataset.

```

glm.pred <- function(dat, mod){
              return(data.frame(pred=predict(mod, newdata = dat), LTV=dat$LTV))}

```

Your output should look as follows:

![embr](./images/embr-4.png)


4. Score Data Using ore.rowApply

Let us first test locally, in open source R first.

```

scores.loc <- glm.pred(dat=cust_insur_ltv_loc, mod=mod.loc)
head(scores.loc)
class(scores.loc)

```

Your output should look as follows:

![embr](./images/embr-5.png)


Now score data using OML4R with database data

```

scores.oml4r <- ore.rowApply(CUST_INSUR_LTV, 
             FUN=glm.pred, 
             mod=mod.loc, 
             rows=200, 
             ore.connect=TRUE, 
             parallel=TRUE,
             FUN.VALUE=data.frame(pred=numeric(0),
                                  LTV=numeric(0)))

class(scores.oml4r)
ore.pull(scores.oml4r)

```

Your output should look as follows:

![embr](./images/embr-6.png)


## Task 8: Conclusion

1.  Conclusion

OML4R enables you to leverage the power of R, a leading statistical programming language, with Oracle Database, the world’s most advanced, and high-performance database. OML4R provides simple, easy to use functions to perform exploratory data analysis including statistical analysis, data visualization, and more advanced and targeted data analysis. OML4R uses overloaded R functions that leverage the powerful Oracle Database to eliminate the need for data movement while utilizing in-database parallel processing.

2. Next Steps

a) Sign-up: OCI Always Free Tier

Consider signing-up for the Oracle Cloud Infrastructure Always Free Tier and use Oracle Machine Learning for R (as well as Oracle Machine Learning for Python and Oracle Machine Learning for SQL) for your development projects. Note that OML4R is currently not supported on Autonomous Database. However, you can use Oracle Database installed in a compute VM and OML4R.

b) Get Certified: Oracle Machine Learning with Oracle Autonomous Database Certification
Consider taking the Oracle Machine Learning with Oracle Autonomous Database Certification offered by Oracle University.


## Acknowledgements
* **Authors** - Ravi Sharma, Rajeev Rumale, Milton Wan
* **Last Updated By/Date** -  Ravi Sharma, 01/24/2022
