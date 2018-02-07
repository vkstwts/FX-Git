Jurka_03_Conway_spam_RTextTools
==================================================================================================================================
## Motivational Buckets
### 1) (size: 99) RTextTools has been used by Jurka to
identify a breast cancer as benign or malignant. However, the predictors are
discrete numbers that are pre-processed into text. Ultimately, we would like to
use RTextTools to predict whether an advertisement is ham (relevant) OR spam
(irrelevant). However, as this is largely an unexplored area, which may result
in low accuracy in prediction, I decided to explore the feasibility of building
a model used to classify large text, i.e. raw text without ANY features.
###
1.1) (size: 81) In the book "Machine Learning for Hackers" by Conway (2012),
there is a chapter devoted to bayesian spam classifier used for email. However,
our interest lies NOT in the bayesian algorithm, but the public database and the
R functions to load the database. Once we have the data loaded, we can build a
prediction model using the RTextTools package.
### 1.2) (size: 99) We compare
the results of our prediction model with Conway's (2012) results, which is
evaluated based on the FALSE-positive (Type I error - false alarm) and FALSE-
negative (Type II error - missed) rates. The FALSE-positive rate is as the
number of outcome positives (outcomes incorrectly classified as spams) divided
by the total conditioned negatives (actual hams), while the FALSE-negative rate
is calculated as the number of outcome negatives (outcomes incorrectly
classified as hams) divided by the total conditioned positives (actual spams).

```{.python .input}
%%R
suppressPackageStartupMessages(require(RTextTools))
suppressPackageStartupMessages(require(tm))
source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE)
spam.dir <- paste0(RegGetRNonSourceDir(), "spamassassin/")
get.msg <- function(path.dir)
{
  con <- file(path.dir, open="rt", encoding="latin1")
  text <- readLines(con)
  msg <- text[seq(which(text=="")[1]+1,length(text),1)]
  close(con)
  return(paste(msg, collapse="\n"))
}
get.msg.try <- function(path.dir)
{
  con <- file(path.dir, open="rt", encoding="latin1")
  text <- readLines(con)
  options(warn=-1)
  msg <- tryCatch( text[seq(which(text=="")[1]+1,length(text),1)],
                      error=function(e) { 9999 }, finally={} )
  close(con)
  if( substr(msg, 1, 5)=="Error" ) 
  {
    return("Error")
  }
  else 
  {
    return(paste(msg, collapse="\n"))
  }
}
get.all <- function(path.dir)
{
  all.file <- dir(path.dir)
  all.file <- all.file[which(all.file!="cmds")]
  msg.all <- sapply(all.file, function(p) get.msg(paste0(path.dir,p)))
}
get.all.try <- function(path.dir)
{
  all.file <- dir(path.dir)
  all.file <- all.file[which(all.file!="cmds")]
  msg.all <- sapply(all.file, function(p) get.msg.try(paste0(path.dir,p)))
}
easy_ham.all    <- get.all(paste0(spam.dir, "easy_ham/"))
easy_ham_2.all  <- get.all(paste0(spam.dir, "easy_ham_2/"))
hard_ham.all    <- get.all(paste0(spam.dir, "hard_ham/"))
hard_ham_2.all  <- get.all(paste0(spam.dir, "hard_ham_2/"))
spam.all        <- get.all.try(paste0(spam.dir, "spam/"))
spam_2.all      <- get.all(paste0(spam.dir, "spam_2/"))
```

### 1.1.1) (size: 35) We explore the raw data, which is the SpamAssassin public
corpus, available for FREE download at
http://spamassassin.apache.org/publiccorpus/. The files are separated into
easy_ham, hard_ham and spam. Of which, EACH set (classification) has TWO (2)
files, e.g. easy_ham and easy_ham_2, which is tarred and bzipped. We will use
the FIRST set for training data, and the SECOND set for testing data.
### 1.1.2)
EACH set resides in its own subfolder within the spamassassin folder, which is
in R-nonsource. For example, the first and second sets for easy_ham resides in
R-nonsource/spamassassin/easy_ham, and R-nonsource/spamassassin/easy_ham_2,
respectively.
### 1.1.3) The raw data include the headers and the message text.
Because we are focusing on ONLY the email message body, we need to extract this
text from the message files. The "null line" separating the header from the body
of an email is part of the protocol definition.
### 1.1.4) We create text
corpuses from EACH set of files by creating a function get.msg() that opens EACH
file, finds the FIRST line break, and returns the text below that break as a
character vector with a SINGLE text element.
### 1.1.5) To create our vector of
messages, we use the get.all() function, which will apply get.msg() function to
ALL of the filenames within a set (specified by a folder) and construct a vector
of messages from the returned text.
### 1.1.6) The assignment statement for
spam.all returns an error when calling the function get.all(). However, it does
NOT return an error when using the get.all.try() function. We may need to
understand why this happens.

```{.python .input}
%%R
head(easy_ham.all)
```

```{.python .input}
%%R
easy_ham.dfr    <- as.data.frame(easy_ham.all)
easy_ham_2.dfr  <- as.data.frame(easy_ham_2.all)
hard_ham.dfr    <- as.data.frame(hard_ham.all)
hard_ham_2.dfr  <- as.data.frame(hard_ham_2.all)
spam.dfr        <- as.data.frame(spam.all)
spam_2.dfr      <- as.data.frame(spam_2.all)
rownames(easy_ham.dfr)    <- NULL
rownames(easy_ham_2.dfr)  <- NULL
rownames(hard_ham.dfr)    <- NULL
rownames(hard_ham_2.dfr)  <- NULL
rownames(spam.dfr)        <- NULL
rownames(spam_2.dfr)      <- NULL
easy_ham.dfr$outcome    <- 2
easy_ham_2.dfr$outcome  <- 2
hard_ham.dfr$outcome    <- 2
hard_ham_2.dfr$outcome  <- 2
spam.dfr$outcome        <- 4
spam_2.dfr$outcome      <- 4
names(easy_ham.dfr)   <- c("text", "outcome")
names(easy_ham_2.dfr) <- c("text", "outcome")
names(hard_ham.dfr)   <- c("text", "outcome")
names(hard_ham_2.dfr) <- c("text", "outcome")
names(spam.dfr)       <- c("text", "outcome")
names(spam_2.dfr)     <- c("text", "outcome")
train.data  <- rbind(easy_ham.dfr, hard_ham.dfr, spam.dfr)
train.num   <- nrow(train.data)
train.data  <- rbind(train.data, easy_ham_2.dfr, hard_ham_2.dfr, spam_2.dfr)
names(train.data) <- c("text", "outcome")
spam.str <- paste0(RegGetRNonSourceDir(),"Jurka_03_spam.rda")
if( !file.exists(spam.str) )
{
  save(train.data, train.num, file=spam.str)
}
```

### 1.1.6) (size: 66) We add a new column "outcome" that classifies BOTH
easy_ham and hard_ham sets to "ham" (2), and spam sets to spam (4). We then
create a data frame "train.data" that contains BOTH the first (training) and
second (testing) sets from EACH classifier. We rename the columns to "text" and
"outcome".
### 1.1.7) We may increase execution speed by saving the processed
data later as an RDA file and to load that file instead. The processed data can
be saved as an RDA file here.

```{.python .input}
%%R
head(train.data)
```

```{.python .input}
%%R
set.seed(2012)
train_out.data <- train.data$outcome
train_txt.data <- train.data$text
```

### 1.1.8) (size: 69) The train.data is also broken into TWO (2) data frames:
(a) train_out.data contains the outcome variable; and (b) train_txt.data
contains ONE (1) predictor, namely "text".
### 1.1.9) The outcome values, which
is a binary outcome consisting of an integer value TWO (2) OR FOUR (4), can be
changed here by just modifying the values in train_out.data. Would the recall
accuracy CHANGE significantly if we modify the outcome values, e.g. ONE (1) and
TWO (2) instead?
### 1.1.10) The predictor values can be changed here by
modifying the values in train_txt.data. We know for instance, from the R
Markdown "Jurka_02_cancer" file, that randomizing missing values (NA) will
increase coverage, but decrease recall significantly.. 
### 1.1.11) For a model
that has a binary outcome, it is suggested that a logistic regression be applied
rather than a linear regression. However, the topic of logistic regression is
beyond the scope of this analysis.

```{.python .input}
%%R
head(train_out.data)
```

```{.python .input}
%%R
matrix <- create_matrix(train_txt.data, language="english", minWordLength=3, removeNumbers=TRUE, stemWords=FALSE, removePunctuation=TRUE, weighting=weightTfIdf)
container <- create_container(matrix,t(train_out.data), trainSize=1:train.num, testSize=(train.num+1):nrow(train.data), virgin=FALSE)
maxent.model    <- train_model(container, "MAXENT")
svm.model       <- train_model(container, "SVM")
#slda.model      <- train_model(container, "SLDA")
#tree.model      <- train_model(container, "TREE")
#bagging.model   <- train_model(container, "BAGGING")
#boosting.model  <- train_model(container, "BOOSTING")
#rf.model        <- train_model(container, "RF")
#nnet.model      <- train_model(container, "NNET")
```

### 1.1.12) (size: 81) The RTextTools package [2] is a wrapper for the tm
package [3]. Therefore, the steps for training a model MUST be as follows: (i)
Create a document-term matrix; (ii) Create a container; (iii) Feed the container
to the machine learning algorithm.
### 1.1.13) To create a matrix, you need to
pass it a either a character vector, e.g. data$Text, or a data frame of columns
containing predictors ONLY. There are some important parameters, such as: (i)
language (default: "english"), (ii) minWordLength (default: 3) - a word should
contain AT LEAST this number of letters to be included in the matrix, (iii)
removeNumbers (default: FALSE) - to specify whether to remove numbers; (iv)
removePunctuation (default: TRUE) - to specify whether to remove punctuations;
(v) stemWords (default: FALSE) - to specify whether to remove stemwords, e.g.
"ing", (vi) weighting (default: weightTf) - this parameter is from the package
tm.
### 1.1.14) For example, to create a matrix using a character vector, e.g.
data$Text, then you should apply the parameters: (i) language="english"; (ii)
minWordLength=3; (iii) removeNumbers=TRUE; (iv) removePunctuation=TRUE; and (v)
stemWords=TRUE. Unfortunately, there is a LIMIT of 255 characters on the number
of characters in a word being stemmed.
### 1.1.15) As another example, if you
allow numbers and punctuations (note: these features were used by Conway (2012)
and Leek (2013) in their text classifiers), then you should apply the following
parameters: (i) language="english"; (ii) minWordLength=3; (iii)
removeNumbers=FALSE; (iv) removePunctuation=FALSE; and (v) stemWords=TRUE.
###
1.1.16) To create a container, you need to pass it BOTH a document-term matrix
AND an outcome vector, e.g. data$Topic. The important parameters are: (i)
trainSize (default: NULL) - a range specifying the row numbers in the matrix to
use for training the model; (ii) testSize (default: NULL) - a range specifying
the row numbers in the matrix to use for out-of-sample testing; (iii) virgin -
to specify whether the testing set is unclassified data with no true value. For
example, when the virgin flag is set to FALSE, indicating that all data in the
training and testing sets have corresponding labels, create_analytics() will
check the results of the learning algorithms against the true value to determine
the accuracy of the process. However, if the virgin flag is set to TRUE,
indicating that the testing set is unclassified data with NO known true value,
create_analytics() will return as much information as possible WITHOUT comparing
EACH predicted value to its true label.
### 1.1.17) To create a model, you need
to pass it a container. The most important parameter is algorithm - a string to
specify which algorithm to use. For expediency, users replicating this analysis
may want to use just the three low-memory algorithms: (i) "SVM" - Support Vector
Machines; (ii) "GLMNET"; and (iii) "MAXENT" - Maximum Entrophy [2]. A
convenience train_models() function trains all models at once by passing in a
vector of model requests, while the print_algorithms() function list ALL NINE
(9) available algorithms.
### 1.1.18) We had to create models separately using
the function train_model(), instead of the train_models() function, as it
returned an error "Error in validObject(.Object) : invalid class "dgRMatrix"
object: slot j is not increasing inside a column". By creating individual
models, we can determine the model that caused this error. We determined that
the error was caused by the algorithm "GLMNET".
### 1.1.19) We also discovered
that the algorithms "SLDA", "TREE", "BAGGING", "BOOSTING", and "RF" requires at
least 4Gb of memory, hence we need a more powerful PC to run these algorithms.
We inadvertantly left out "NNET" algorithm.
### 1.1.20) In our first attempt, we
created models using BOTH algorithms "MAXENT" and "SVM", and analyzed their
results.

```{.python .input}
%%R
svm.result    <- classify_model(container, svm.model)
svm.analytic  <- create_analytics(container, svm.result)
svm.doc       <- svm.analytic@document_summary
svm_spam.doc  <- svm.doc[svm.doc$MANUAL_CODE==4, ]
svm_ham.doc   <- svm.doc[svm.doc$MANUAL_CODE==2, ]
svm.true.pos  <- nrow(svm_spam.doc[svm_spam.doc$CONSENSUS_CODE==4,]) / nrow(svm_spam.doc)
svm.false.neg <- nrow(svm_spam.doc[svm_spam.doc$CONSENSUS_CODE==2,]) / nrow(svm_spam.doc)
svm.true.neg  <- nrow(svm_ham.doc[svm_ham.doc$CONSENSUS_CODE==2,]) / nrow(svm_ham.doc)
svm.false.pos <- nrow(svm_ham.doc[svm_ham.doc$CONSENSUS_CODE==4,]) / nrow(svm_ham.doc)
maxent.result   <- classify_model(container, maxent.model)
maxent.analytic <- create_analytics(container, maxent.result)
maxent.doc      <- maxent.analytic@document_summary
maxent_spam.doc <- maxent.doc[maxent.doc$MANUAL_CODE==4, ]
maxent_ham.doc  <- maxent.doc[maxent.doc$MANUAL_CODE==2, ]
maxent.true.pos <- nrow(maxent_spam.doc[maxent_spam.doc$CONSENSUS_CODE==4,]) / nrow(maxent_spam.doc)
maxent.false.neg<- nrow(maxent_spam.doc[maxent_spam.doc$CONSENSUS_CODE==2,]) / nrow(maxent_spam.doc)
maxent.true.neg <- nrow(maxent_ham.doc[maxent_ham.doc$CONSENSUS_CODE==2,]) / nrow(maxent_ham.doc)
maxent.false.pos<- nrow(maxent_ham.doc[maxent_ham.doc$CONSENSUS_CODE==4,]) / nrow(maxent_ham.doc)
```

### 1.2.1) (size: 99) The functions classify_model() and classify_models() use
the same syntax as train_model(). Each model created in the previous step is
passed on to classify_model(), which then returns the classified data.
###
1.2.2) The function create_analytics() returns a container with FOUR (4)
different summaries: (i) ensemble (consensus) - refers to whether multiple
algorithms make the same prediction concerning the classification; (ii)
algorithm - provides a breakdown of EACH algorithm's performance for each unique
label in the classified data, e.g. EACH topic category (outcome); (iii) label -
provides statistics for EACH unique label in the classified data; and (iv)
document - provides ALL the raw data available for each document, including EACH
algorithm's prediction.
### 1.2.3) Precision refers to how often a case the
algorithm predicts as belonging to a class actually belongs to that class. For
example, in the context of the USCongress data, precision tells us what
proportion of bills an algorithm deems to be about defense are actually about
defense(based on the human-assigned labels).
### 1.2.4) In contrast, recall
refers to the proportion of bills in a class the algorithm correctly assigns to
that class. For example, what percentage of actual defense bills did the
algorithm correctly classify?
### 1.2.5) F-scores produce a weighted average of
both precision and recall, where the highest level of performance is equal to
ONE (1) and the lowest ZERO (0).
### 1.2.6) Coverage simply refers to the
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
### 1.2.7) We compare our
results with Conway's (2012) results, which are shown in a table below. In order
to compare the results, we have to calculate the values for EACH column. 
####
1.2.7.1) In the first column (TRUE): (i) spam: sum of outcome positives divided
by total spams; (ii) easy_ham: sum of outcome negatives divided by total
easy_hams; (iii) hard_ham: sum of outcome negatives divided by total hard_hams.
#### 1.2.7.2) In the second column (FALSE): (i) spam: sum of outcome negatives
divide by total spams; (ii) easy_ham: sum of outcome positives divided by total
easy_hams; (iii) hard_ham: sum of outcome positives divided by total hard_hams.
#### Email Type | TRUE          | FALSE
#### spam       | T-Pos: 85%    | F-Neg:
15%
#### easy_ham   | T-Neg: 78%    | F-Pos: 22%
#### hard_ham   | T-Neg: 73%
| F-Pos: 27%
### 1.2.8) Our results using SVM algorithm:
#### Email Type | TRUE
| FALSE
#### Spam       | T-Pos: 86.8%  | F-Neg: 13.2%
#### Ham        | T-Neg:
96.8%  | F-Pos: 3.2%
### 1.2.9) Our results using MAXENT algorithm:
#### Email
Type | TRUE          | FALSE
#### Spam       | T-Pos: 85.3%  | F-Neg: 14.7%
####
Ham        | T-Neg: 99.6%  | F-Pos: 0.4%



## References
### Jurka, RTextTools:
a machine learning library for text classification. URL: www.rtexttools.com.
Accessed on 18-Feb-2013.
### Jurka et al (2012), RTextTools: A Supervised
Learning Package for Text Classification.
### Feinerer, K. Hornik and D. Meyer.
Text Mining Infrastructure in R. Journal of Statistical Software, 25 (5). 2008.
URL www.jstatsoft.org/v25/i05.
### Conway (2012), Machine Learning for Hackers.
