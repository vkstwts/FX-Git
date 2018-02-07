Job_01_classify_RTextTools
==================================================================================================================================
## Motivational Buckets
### 1) (size: ) This is a classification feasibility
study. While there are many methods to classify text, we are inspired by the
RTextTools breast cancer classification (see the R Markdown file
Jurka_02_cancer_RTextTools) to use it for classification of jobs.
### 1.1)
(size: ) There are TWO (2) elements that are required: (a) text to be
classified; and (b) manually-coded classification (topic). For example, a breast
cancer is classified as EITHER benign (2) OR active (4).

```{.python .input}
%%R
source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE)
source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusFile.R", echo=FALSE)
source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusJob.R", echo=FALSE)
suppressPackageStartupMessages(require(RTextTools))
job.str <- paste0(RegGetRNonSourceDir(), "jobEfc_Trading.csv")
if( !file.exists(job.str) ) jobEfcUpdateNum("Trading")
jobDfr <- fileReadDfr("jobEfc_Trading")
```

### 1.1.1) (size: ) First load the data file, if exists. Otherwise, we call the
function jobEfcUpdateNum(). 
### 1.1.2) The function jobEfcUpdateNum() scrapes
the relevant job details from the web page and appends the data into am existing
CSV file (OR creates a new CSV file, if it does NOT exist). The most important
parameter is sectorStr - a string to specify EITHER "Accounting_Finance",
"Asset_Management", "Capital_Markets", "Commodities", "Equities",
"FX_Money_Markets", "Hedge_Funds", "Quantitative_Analytics", "Research", OR
"Trading".
### 1.1.3) The raw data contains the first element, i.e. text to be
classified, in the column "content". However, the second element has to be
MANUALLY encoded, as a new column "topic". It is unclear whether topic should be
either: (a) a binary outcome; OR (b) a discrete outcome. As it is easier to
reduce the outcome later, we select topic to be a discrete outcome. 
### 1.1.4)
The discrete outcome "topic" has the following values: (i) 0000 - NA; (ii) 0001
- trader; (iii) 0010 - analyst; (iv) 0011 - fund manager; (v) 0100 - sell-side;
(vi) 1000 - buy-side; (vii) 01100 - quantitative; (viii) 1101 - technical; (ix)
1110 - fundamental. For example, a proprietary trading based on technical
analysis job is 1+8+13=22, while a quantitative sell-side analyst job is
2+4+12=18. It is imperative that ALL possible combinations are unique. However,
it is unclear if discrete outcomes with unique combinations yields better
results than binary outcome.
### 1.1.5) To manually encode: (i) Is it a trader,
analyst, fund manager OR other? Add 1, 2, 3 OR 0 respectively; (ii) Is it a
sell-side, buy-side OR neither? Add 4, 8 OR 0 respectively; (iii) Is it a
quantitative, technical, fundamental OR none? Add 12, 13, 14 OR 0 respectively.
## References
