Jurka_02_cancer_RTextTools
==================================================================================================================================
## Motivational Buckets
### 1) (size: 22) RTextTools has largely been used for
topic classification in the social sciences. However, recent applications at
various universities have demonstrated (ED: to a smaller extent) that the
package can be applied to a host of problems in the natural sciences as well
[1].
### 1.1) (size: 13) Jurka wrote a script that trains NINE (9) classifiers
on characteristics such as clump thickness, uniformity of cell size, uniformity
of cell shape, marginal adhesion, single epithelial cell size, bare nuclei,
bland chromatin, normal nucleoli, and mitoses. The outcome variable is the LAST
column of the data frame, which is used to identify a breast cancer as benign or
malignant.
### 1.2) (size: 22) When run on the data, the classifiers were able
to achieve up to 96% recall accuracy on a randomly sampled training set of 200
patients and test set of 400 patients.

```{.python .input}
%%R
suppressPackageStartupMessages(require(RTextTools))
data <- read.csv("http://archive.ics.uci.edu/ml/machine-learning-databases/breast-cancer-wisconsin/breast-cancer-wisconsin.data", header=FALSE)
```

### 1.1.1) (size: 2) We explore the raw data, which is the Wisconsin Diagnostic
Breast Cancer Dataset from UC Irvine. The file is downloaded EACH time we run
the R script. However, we may increase execution speed by saving the processed
data later as an RDA file and to load that file instead.
### 1.1.2) We note that
there are ELEVEN (11) columns in the raw data, but ONLY columns TWO (2) to
ELEVEN (11) are used in our analysis.

```{.python .input}
%%R
names(data)
head(data)
```

```{.python .input}
%%R
data <- data[-1]
thick <- as.vector(apply(as.matrix(data[1], mode="character"),1,paste,"clump",sep="",collapse=""))
size <- as.vector(apply(as.matrix(data[2], mode="character"),1,paste,"size",sep="",collapse=""))
shape <- as.vector(apply(as.matrix(data[3], mode="character"),1,paste,"shape",sep="",collapse=""))
adhesion <- as.vector(apply(as.matrix(data[4], mode="character"),1,paste,"adhesion",sep="",collapse=""))
single <- as.vector(apply(as.matrix(data[5], mode="character"),1,paste,"single",sep="",collapse=""))
nuclei <- as.vector(apply(as.matrix(data[6], mode="character"),1,paste,"nuclei",sep="",collapse=""))
chromatin <- as.vector(apply(as.matrix(data[7], mode="character"),1,paste,"chromatin",sep="",collapse=""))
nucleoli <- as.vector(apply(as.matrix(data[8], mode="character"),1,paste,"nucleoli",sep="",collapse=""))
mitoses <- as.vector(apply(as.matrix(data[9], mode="character"),1,paste,"mitoses",sep="",collapse=""))
training_data <- cbind(data[10],thick,size,shape,adhesion,single,nuclei,chromatin,nucleoli,mitoses)
```

### 1.1.3) (size: 13) The processed data can be saved as an RDA file here, after
transforming the numerical columns into characters.
### 1.1.4) It is apparent
that the predictors used are first converted into character by appending a text
to EACH numerical value. The reason I can think of is that it is either (a) a
limitation of the RTextTools package; or (b) it is more robust and efficient to
do so using the package.

```{.python .input}
%%R
source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE)
cancer.str <- paste0(RegGetRNonSourceDir(),"Jurka_02_cancer.rda")
if( !file.exists(cancer.str) )
{
  save(training_data, file=cancer.str)
}
head(training_data)
```

```{.python .input}
%%R
training_data <- training_data[sample(1:699,size=600,replace=FALSE),]
training_codes <- training_data[1]
training_data <- training_data[-1]
```

### 1.2.1) (size: 16) First, to ensure a reproducible research, the function
set.seed() should be used. However, the author does NOT use this function before
sampling 600 rows from the data. An issue here is why didn't the author just use
ALL 699 rows in the data? The only reason I can think of is that he is applying
the bootstrapping method, however this method requires sampling WITH
replacement.
### 1.2.2) The sample of 600 rows is also broken into TWO (2) data
frames: (a) training_codes contains the outcome variable; and (b) training_data
contains NINE (9) predictors.
### 1.2.3) The outcome values can be changed here
by just modifying the values in training_codes, which is a binary outcome
consisting of an integer value TWO (2) OR FOUR (4). Would the recall accuracy
DECREASE if we modify the outcome values?
### 1.2.4) The predictor values can be
changed here by modifying the values in training_data. We could for example,
randomizing missing values (NA) and observe how the models would handle missing
data? 
### 1.2.5) For a model that has a binary outcome, it is suggested that a
logistic regression be applied rather than a linear regression. However, the
topic of logistic regression is beyond the scope of this analysis.

```{.python .input}
%%R
class(training_codes)
head(training_codes)
class(training_data)
head(training_data)
table(training_codes)
tMissingData <- training_data[1]
for( i in 1:10 )
{
  r <- sample(1:nrow(tMissingData), size=1, replace=FALSE)
  c <- sample(1:ncol(tMissingData), size=1, replace=FALSE)
  tMissingData[r,c] <- NA
}
```

```{.python .input}
%%R
matrix <- create_matrix(training_data, language="english", removeNumbers=FALSE, stemWords=FALSE, removePunctuation=FALSE, weighting=weightTfIdf)
container <- create_container(matrix,t(training_codes),trainSize=1:200, testSize=201:600,virgin=FALSE)
models <- train_models(container, algorithms=c("MAXENT","SVM","GLMNET","SLDA","TREE","BAGGING","BOOSTING","RF"))
```

### 1.2.5) (size: 19) The RTextTools package [2] is a wrapper for the tm package
[3]. Therefore, the steps for training a model MUST be as follows: (i) Create a
document-term matrix; (ii) Create a container; (iii) Feed the container to the
machine learning algorithm.
### 1.2.6) To create a matrix, you need to pass it a
either a character vector, e.g. data$Text, or a data frame of columns containing
predictors ONLY. There are some important parameters, such as: (i) language
(default: "english"), (ii) minWordLength (default: 3) - a word should contain AT
LEAST this number of letters to be included in the matrix, (iii) removeNumbers
(default: FALSE) - to specify whether to remove numbers; (iv) removePunctuation
(default: TRUE) - to specify whether to remove punctuations; (v) stemWords
(default: FALSE) - to specify whether to remove stemwords, e.g. "ing", (vi)
weighting (default: weightTf) - this parameter is from the package tm.
###
1.2.7) For example, to create a matrix using a character vector, e.g. data$Text,
then you should apply the parameters: (i) language="english"; (ii)
minWordLength=3; (iii) removeNumbers=TRUE; (iv) removePunctuation=TRUE; and (v)
stemWords=TRUE.
### 1.2.8) As another example, should you pass it a hand-coded
data frame, e.g. breast cancer, then you should apply the following parameters:
(i) language="english"; (ii) minWordLength=3; (iii) removeNumbers=FALSE; (iv)
removePunctuation=FALSE; and (v) stemWords=FALSE.
### 1.2.9) To create a
container, you need to pass it BOTH a document-term matrix AND an outcome
vector, e.g. data$Topic. The important parameters are: (i) trainSize (default:
NULL) - a range specifying the row numbers in the matrix to use for training the
model; (ii) testSize (default: NULL) - a range specifying the row numbers in the
matrix to use for out-of-sample testing; (iii) virgin - to specify whether the
testing set is unclassified data with no true value. For example, when the
virgin flag is set to FALSE, indicating that all data in the training and
testing sets have corresponding labels, create_analytics() will check the
results of the learning algorithms against the true value to determine the
accuracy of the process. However, if the virgin flag is set to TRUE, indicating
that the testing set is unclassified data with NO known true value,
create_analytics() will return as much information as possible WITHOUT comparing
EACH predicted value to its true label.
### 1.2.10) To create a model, you need
to pass it a container. The most important parameter is algorithm - a string to
specify which algorithm to use. For expediency, users replicating this analysis
may want to use just the three low-memory algorithms: (i) "SVM" - Support Vector
Machines; (ii) "GLMNET"; and (iii) "MAXENT" - Maximum Entrophy [2]. A
convenience train_models() function trains all models at once by passing in a
vector of model requests, while the print_algorithms() function list ALL NINE
(9) available algorithms.

```{.python .input}
%%R
missing.dtm <- create_matrix(tMissingData, language="english", removeNumbers=FALSE, stemWords=FALSE, removePunctuation=FALSE, weighting=weightTfIdf)
missing.ctn <- create_container(missing.dtm, t(training_codes),trainSize=1:200, testSize=201:600,virgin=FALSE)
missing.models <- train_models(missing.ctn, algorithms=c("MAXENT","SVM","GLMNET","SLDA","TREE","BAGGING","BOOSTING","RF"))
```

```{.python .input}
%%R
results <- classify_models(container, models)
analytics <- create_analytics(container, results)
analytics@ensemble_summary
```

### 1.2.11) (size: 22) The functions classify_model() and classify_models() use
the same syntax as train_model(). Each model created in the previous step is
passed on to classify_model(), which then returns the classified data.
###
1.2.12) The function create_analytics() returns a container with FOUR (4)
different summaries: (i) ensemble (consensus) - refers to whether multiple
algorithms make the same prediction concerning the classification; (ii)
algorithm - provides a breakdown of EACH algorithm's performance for each unique
label in the classified data, e.g. EACH topic category (outcome); (iii) label -
provides statistics for EACH unique label in the classified data; and (iv)
document - provides ALL the raw data available for each document, including EACH
algorithm's prediction.
### 1.2.13) Precision refers to how often a case the
algorithm predicts as belonging to a class actually belongs to that class. For
example, in the context of the USCongress data, precision tells us what
proportion of bills an algorithm deems to be about defense are actually about
defense(based on the human-assigned labels).
### 1.2.14) In contrast, recall
refers to the proportion of bills in a class the algorithm correctly assigns to
that class. For example, what percentage of actual defense bills did the
algorithm correctly classify?
### 1.2.15) F-scores produce a weighted average of
both precision and recall, where the highest level of performance is equal to
ONE (1) and the lowest ZERO (0).
### 1.2.16) Coverage simply refers to the
percentage of documents that meet the recall accuracy threshold. For instance,
say we find that when seven algorithms agree on the label of a bill, our overall
accuracy is 90% (when checked against our true values). Then, let's say, we find
that only 20% of our bills meet that criterion. If we have 10 bills and only two
bills meet the seven ensemble agreement threshold, then our coverage is 20%.
Mathematically, if k represents the percent of cases that meet the ensemble
threshold, and n represents total cases, coverage is calculated as k/n. The
general trend is for coverage to decrease while recall increases. For example,
just 11% of the congressional bills in our data have nine algorithms that agree.
However, recall accuracy is 100% for those bills when the 9 algorithms do agree.
Considering that 90% is often social scientists' inter-coder reliability
standard, one may be comfortable using a 6 ensemble agreement with these data
because we label 66% of the data with accuracy at 90%.
### 1.2.17) By
randomizing TEN (1) missing values (NA) into the predictors, we note that
coverage has increased but recall has decreased significantly.

```{.python .input}
%%R
doc <- analytics@document_summary
doc[doc$MANUAL_CODE != doc$CONSENSUS_CODE, 17:18]
missing.results <- classify_models(missing.ctn, missing.models)
missing.analytics <- create_analytics(missing.ctn, missing.results)
missing.analytics@ensemble_summary
```

## References
### Jurka, RTextTools: a machine learning library for text
classification. URL: http://www.rtexttools.com/. Accessed on 18-Feb-2013
###
Jurka et al (2012), RTextTools: A Supervised Learning Package for Text
Classification.
### Feinerer, K. Hornik and D. Meyer. Text Mining Infrastructure
in R. Journal of Statistical Software, 25
(5). 2008. URL
http://www.jstatsoft.org/v25/i05/.
