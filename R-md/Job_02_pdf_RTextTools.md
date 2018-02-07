Job_02_pdf_RTextTools
==================================================================================================================================
## Motivational Buckets
### 1) (size: 75) This project is the SECOND application
of the RTextTools package. The motivation to build this research report (PDF)
classifier prediction model is a result of the "Jurka_03_Conway_spam_RTextTools"
feasibility study, which was beyond my expectation. The first task is to build a
database of PDFs to be classified. Ultimately, we would like to use RTextTools
to predict whether a PDF is ham (relevant) OR spam (irrelevant). Previously, we
had explored the feasibility of building a model used to classify large text,
i.e. raw text without ANY features. However, we may decide to implement features
at a later stage.
### 1.1) (size: 12) As there are NO PDF databases, there are
several steps prior to loading the data: (i) If the RDA file exists, load it;
(ii) Scrape the data from a web page into memory; (iii) If the data frame
exists, append the data-in-memory to the data frame, otherwise create a new data
frame; (iv) Save ALL the data frames (including existing and NEW ones) into the
RDA file. 
### 1.2) (size: 74) Once we have the data loaded, we can build a
prediction model using ONLY the rows that have a GOLD standard, i.e. manual code
classification (gold data). There are several steps prior to using our
prediction model: (i) Split the gold data into the train/test sets; (ii) If the
RDA file exists, load it; (iii) Create an analytics model, which may be
evaluated based on the FALSE-positive (Type I error - false alarm) and FALSE-
negative (Type II error - missed) rates, and compare the results to our previous
analytics model; (iii) If the new model is "above expectation", supercede the
older model with it; (iv) Save ALL the models (excluding model that is "below or
meet expectation") into the RDA file.
### 1.3) (size: 75) Once we have the model
loaded, we can allow the model to predict outcomes, i.e. classify a job
advertisement as EITHER ham OR spam.

```{.python .input}
%%R
if( Sys.info()["sysname"] == "Linux" )
{
  source("~/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE)
  source("~/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusFile.R", echo=FALSE)
  source("~/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusPdf.R", echo=FALSE)
  source("~/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusRtt.R", echo=FALSE)
}
if( Sys.info()["sysname"] == "Windows" )
{
  source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE)
  source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusFile.R", echo=FALSE)
  source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusPdf.R", echo=FALSE)
  source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusRtt.R", echo=FALSE)
}
file.str <- paste0(RegGetRNonSourceDir(), "Job_02_pdf.rda")
pred.str <- paste0(RegGetRNonSourceDir(), "Job_02_mdl.rda")
```

### 1.1.1) (size: 12) We had to scrape the PDFs from the web page with the
function PdfNomuraSeqNum() using default parameters except for the following
parameters: (i) toNum=5 - number of PDFs to download; (ii)
toChr="dennislwm@yahoo.com.au" - email address of recipient; (iii) waitNum=20 -
number of seconds to wait between EACH query; (iv) silent=TRUE - do NOT display
console messages; (v) fileStr="Job_02_pdf.rda" - full pathname of the RDA file.
### 1.1.2) At ANY stage, the RDA file should contain a data frame "pdfDfr",
which consists of virgin data as well as train/test data. The function
PdfNomuraSeqNum() loads fileStr at start and appends ALL new rows to the
existing "pdfDfr". However, if this is the first time that fileStr is used, then
the function will create a new fileStr and a new "pdfDfr".
### 1.1.3) The data
frame "pdfDfr" consists of only TWO (2) columns: (a) text; (b) outcome, and it
contains ALL the training, testing and virgin data. Virgin data is ANY appended
data, i.e. a new row consisting of the text, which is the first TWENTY (20)
lines of a PDF, and the outcome is the default value NA, which makes it is easy
to split the virgin data from the train/test data.

```{.python .input}
%%R
PdfTrainPlanDfr <- function(fileStr, predStr=NULL, setNum=1, train.range=NULL, minSize=10)
{
  load(fileStr)
  if( !exists("pdfDfr") ) 
    return( FALSE );
  datDfr <- pdfDfr[ !is.na(pdfDfr$outcome), ]
  if( nrow(datDfr) < minSize * 2 ) 
    return( FALSE );
  if( as.numeric(setNum) < 1 )
    stop("setNum MUST be GREATER than OR equal to ONE (1)")
  if( as.numeric(setNum) > 100 )
    stop("setNum MUST be LESS than OR equal to ONE HUNDRED (100)")
  if( as.numeric(minSize) < 1 )
    stop("minSize MUST be GREATER than OR equal to ONE (1)")
  
  train.min     <- minSize
  min.false.neg <- 1
  min.false.pos <- 1
  
  #---  Assert return data frame
  optDfr <- dataFrame( colClasses=c( trainNum="numeric", seedNum="numeric",
                                     false.neg="numeric", false.pos="numeric"), nrow=0 )
  #---  Assert save file  
  saveBln <- !is.null(predStr)
  if( saveBln )
  {
    doBln <- FALSE
    if( !file.exists(predStr) )
      doBln <- TRUE
    else
      load(predStr)
    if( !exists("optDfr") )
      doBln <- TRUE
    if( doBln )
    {
      save( optDfr, file=predStr )  
    }
    if( nrow(optDfr) > 0 )
    {
      train.min     <- as.numeric( optDfr$trainNum[1] )
      min.false.neg <- as.numeric( optDfr$false.neg[1] )
      min.false.pos <- as.numeric( optDfr$false.pos[1] )
    }
  }
  
  if( is.null(train.range) )
  {
    train.max <- nrow(datDfr) - minSize
    if( train.min > train.max)
      return( FALSE )
    else
      train.range <- train.min:train.max
  }
  
  found <- FALSE
  for( t in train.range )
  {
    for( seq in 1:setNum )
    {
      pdf.seed  <- trunc(runif(1)*10000)
      pdf.ctn   <- RttTrainPlan.ctn(datDfr, t, seedNum=pdf.seed)
      pdf.mdl   <- RttTrainAct.mdl(pdf.ctn$container)
      do.1      <- pdf.mdl$false.neg < min.false.neg
      do.2      <- pdf.mdl$false.pos < min.false.pos
      if( do.1 & do.2 )
      {
        found         <- TRUE
        opt.trainNum  <- t
        opt.seedNum   <- pdf.seed
        opt.false.neg <- pdf.mdl$false.neg
        opt.false.pos <- pdf.mdl$false.pos
      }
    }
  }
  
  if( found )
  {
    opt.names     <- names(optDfr)
    optDfr        <- rbind(data.frame(trainNum=opt.trainNum, seedNum=opt.seedNum,
                                      false.neg=opt.false.neg, false.pos=opt.false.pos),
                           optDfr)
    names(optDfr) <- opt.names
  }
  if( saveBln )
    save( optDfr, file=predStr )  
  optDfr
}
```

### 1.2.1) (size: 74) The function PdfTrainPlanDfr() accepts TWO (2) parameters:
(i) fileStr - a string for a full path to the RDA file containing the raw data
"pdfDfr"; (ii) predStr - a string for a full path to the RDA file containing a
data frame "optDfr".
### 1.2.2) At ANY stage, the "pred.str" RDA file should
contain the data frame "optDfr", which consists of FOUR (4) columns: (i)
"trainNum" - an integer for the sample size used in the training model
(excluding test model); (ii) "seedNum" - an integer for the random seed; (iii)
"false.neg" - a numeric for the false negative rate; and (iv) "false.pos" - a
numeric for the false positive rate.

```{.python .input}
%%R
PdfTrainPlanDfr(file.str, pred.str, setNum=10)
```

### 1.3.1) (size: 75) The function PdfTrainPlanDfr() loads "pred.str" at start
(if NOT NULL), and sets the minimum trainNum, false.neg and false.pos to the
FIRST row values in the data frame "optDfr". This function then iterates through
"setNum" times for EACH trainNum (ranging from minimum TO nrow(data)-minSize )
to find optimal values of "false.neg" and "false.pos". If found, it appends the
relevant trainNum, seedNum and optimal values into the FOUR (4) columns as the
FIRST row in "optDfr", otherwise NO row is appended. It then saves the data
frame to "pred.str" (if NOT NULL), and returns the data frame "optDfr".

##
References
### Jurka, RTextTools: a machine learning library for text
classification. URL: www.rtexttools.com. Accessed on 18-Feb-2013.
### Jurka et
al (2012), RTextTools: A Supervised Learning Package for Text Classification.
### Feinerer, K. Hornik and D. Meyer. Text Mining Infrastructure in R. Journal
of Statistical Software, 25 (5). 2008. URL www.jstatsoft.org/v25/i05.
### Conway
(2012), Machine Learning for Hackers.
