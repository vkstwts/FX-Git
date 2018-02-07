Lotto_02_aic_forecast
==================================================================================================================================
## Motivational Buckets
### 1) (size: 56) This study is NOT just about the model
accuracy and prediction, BUT also the selection process for new estimates. We
MUST identify ANY deficiency in the current models, compare them with new
prospective models, and understand WHY one model is "better" than another. Also,
preference is given to the application, analysis and synthesis that are
difficult to perform using the Shiny package.
### 1.1) (size: 7) I do NOT quite
understand why a diff.arima model is "better" at predicting than a
log.diff.arima model. By "better", I mean TWO (2) things: (a) confidence
intervals are tighter; (b) actual results lie within confidence intervals.
###
1.2) (size: 37) Also, we MUST size up, weigh and value EACH number in an
estimate. We should determine if the size, weight OR value of a number has
changed with respective to predicted estimates and actual results. 
### 1.3)
(size: 56) Historically, some plots would make the data more interesting: (i)
the actual number vs confidence interval should be plotted for past number of
draws.

```{.python .input}
%%R
source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusLotto.R", echo=FALSE)
lotto.str <- "toto"
if( ask("Update ALL Lotto results (y OR <blank> for n)?")=="y" ) LottoUpdateNum()
factorial(45)/factorial(38)/lottoArimaNum(lotto.str, 2, FALSE)
factorial(45)/factorial(38)/lottoArimaNum(lotto.str, 2, TRUE)
LottoArimaConf(lotto.str, 2, TRUE)
LottoResult(lotto.str)
```

### 1.1.1) (size: 7) We must ensure that the files contain the latest results.
First, we check the predicted confidence intervals vs the actual results.
###
1.1.2) The 80% confidence interval means that if a sample were taken, then the
probability of a true value lying within that confidence interval is 80%. In
other words, there is a 20% chance that the confidence interval is off the mark.
Coincidentally, for toto Draw No. 2824, EVERY number (including SUM) were within
their respective 80% confidence intervals.

```{.python .input}
%%R
result.cex <- function(result, past) 
{
  past.table <- table(as.numeric(past))
  result.exist <- length(which(names(past.table)==result))>0
  if( result.exist )
  {
    table.col <- which(names(past.table)==result)
    past.table[[table.col]] / sum(past.table)
  } else
  {
    0
  }
}
rawDfr <- fileReadDfr(lotto.str)
conf.list <- list()
for( startNum in 1:10 )
{
  conf.list[[startNum]] <- LottoArimaConf(lotto.str, startNum, TRUE)
}
par(mar=c(4,4,0,2), mfrow=c(3,3))
for( rollNum in 1:7 )
{
  plot(1:10,type="n",xlim=c(1,45),ylim=c(1,10), xlab="Confidence Intervals",ylab="StartNum")
  for( startNum in 1:10 )
  {
    confDfr <- conf.list[[startNum]]
    lowerNum <- confDfr[rollNum, 1]
    upperNum <- confDfr[rollNum, 2]
    segments(lowerNum, startNum, upperNum, startNum, col='grey')
    if( startNum > 1 )
    {
      resultNum <- as.numeric(rawDfr[ startNum-1, rollNum+2])
      resultCex <- result.cex(resultNum, as.numeric(rawDfr[, rollNum+2])) / (1/45)
      if( resultNum >= lowerNum & resultNum <= upperNum )
      {
        points(resultNum, startNum, col='grey', cex=resultCex)
      }
      else
      {
        points(resultNum, startNum, col='red', pch=19, cex=resultCex)
      }
    }
  }
}

factorial(45)/factorial(38)/lottoArimaNum(lotto.str, 1, FALSE)
factorial(45)/factorial(38)/lottoArimaNum(lotto.str, 1, TRUE)
LottoArimaConf(lotto.str, 1, TRUE)
```

### 1.2.1) (size: 37) Historically (Last Draw: 2825), there were SEVEN (7)
incorrect confidence intervals in the last NINE (9) draws (excluding additional
number). This works out to 7/54 (or 13.0%) error, which is significantly below
the 20% error rate. Of which, FIVE (5) draws had ZERO (0) error, ONE (1) draw
had ONE (1) error, and THREE (3) draws had TWO (2) errors. This means that the
chance of at least ONE (1) error in a draw is 44.4%, and at least TWO (2) errors
in a draw is 33.3%.
### 1.2.2) If we look at individual digits (columns) in the
last NINE (9) draws (excluding additional number), digit #1 had THREE (3) errors
(33.3%), digit #2 had THREE (3) errors (33.3%), digit #3 had ONE (1) error
(11.1%), digit #4 to #6 had ZERO (0) error (0.0%).
### 1.2.3) Next, we rank the
digits based on eye-balling the errors and intervals, from most likely to be
correct as the highest: digit #6, #5, #4, #3, #1, #2. Hence this results in the
confidence intervals being ranked as follows:
#### lower - upper
#### 2: 06 - 36
#### 1: 03 - 35
#### 3: 11 - 40
#### 4: 14 - 42
#### 5: 19 - 44
#### 6: 25 - 45
### 1.2.4) Based on Last Draw: 2826, there were SIX (6) incorrect confidence
intervals (excluding additional number). This decreases the error rate from
13.0% to 11.1% (8/54). Of which, FIVE (5) draws had ZERO (0) error, TWO (2)
draws had ONE (1) error, and  TWO (2) draws had TWO (2) errors. This means that
the chance of ZERO (0) error in a draw remained the same at 55.5%, at least ONE
(1) error in a draw remained the same at 44.4%, and at least TWO (2) errors in a
draw has decreased from 33.3% to 22.2%.
### 1.2.5) Looking at individual digits
(excluding additional number), digit #1 had the same error rate at 33.3%, digit
#2 error rate has decreased from 33.3% to 22.2%, digit #3 had the same error
rate at 11.1%, digit #4 to #6 had EACH maintained the same error rate at 0.0%.
### 1.2.6) Prediction model has to forecast the expected trend, not historical
trend. Simplistically, what has gone up MUST come down, and vice-versa. In this
case, we are only interested in digit #2 as this has decreased historically,
which means that it will increase in the future. Also, we are interested in at
least TWO (2) errors in a draw as the error rate had decreased historically,
therefore we expect it to increase in the future.
### 1.2.7) Next, we rank the
digits based on eye-balling the errors, and using our simplistic prediction
model, from most likely to be INCORRECT TOP THREE (3):
#### lower - upper
(INCORRECT TOP 3)
#### 2: 02 - 17
#### 3: 05 - 21
#### 4: 12 - 37
#### lower -
upper (CORRECT TOP 3)
#### 6: 27 - 45
#### 1: 01 - 24
#### 5: 17 - 40

```{.python .input}
%%R
conf.list <- list()
for( startNum in 1:10 )
{
  conf.list[[startNum]] <- LottoArimaConf(lotto.str, startNum, FALSE)
}
par(mar=c(4,4,0,2), mfrow=c(3,3))
for( rollNum in 1:7 )
{
  plot(1:10,type="n",xlim=c(1,45),ylim=c(1,10), xlab="Confidence Intervals",ylab="StartNum")
  for( startNum in 1:10 )
  {
    confDfr <- conf.list[[startNum]]
    lowerNum <- confDfr[rollNum, 1]
    upperNum <- confDfr[rollNum, 2]
    segments(lowerNum, startNum, upperNum, startNum, col='grey')
    if( startNum > 1 )
    {
      resultNum <- as.numeric(rawDfr[ startNum-1, rollNum+2])
      resultCex <- result.cex(resultNum, as.numeric(rawDfr[, rollNum+2])) / (1/45)
      if( resultNum >= lowerNum & resultNum <= upperNum )
      {
        points(resultNum, startNum, col='grey', cex=resultCex)
      }
      else
      {
        points(resultNum, startNum, col='red', pch=19, cex=resultCex)
      }
    }
  }
}
LottoArimaConf(lotto.str, 1, FALSE)
```

### 1.3.1) (size: 56) The first set assumes that at least TWO (2) intervals will
be incorrect, which has a 33.3% chance historically. We assume digit #2 and
digit #3 to be incorrect.
#### 1.3.1.1) sys7: 9, 26, 3, 13, 34, 38, 41
####
1.3.1.2) sys7: 13, 17, 22, 14, 31, 40, 42
#### 1.3.1.3) sys7: 4, 23, 28, 21, 37,
39, 41
#### 1.3.1.4) sys7: 8, 31, 33, 17, 20, 35, 44
### 1.3.2) The first set
assumes that at least TWO (2) intervals will be incorrect, which has a 33.3%
chance historically. We assume digit #4 and digit #5 to be incorrect.
####
1.3.2.1) sys7: 6, 8, 14, 39, 42, 40, 34
#### 1.3.2.2) sys7: 9, 12, 10, 41, 43,
45, 30
#### 1.3.2.3) sys7: 4, 3, 6, 38, 42, 37, 36
#### 1.3.2.4) sys7: 12, 11,
18, 10, 16, 44, 42
### 1.3.3) The second set assumes that ONE (1) interval will
be incorrect, which has a 11.1% chance historically. We assume digit #2 to be
incorrect. This set is NOT required as it is an in-between set.
### 1.3.4) The
third set assumes that ALL intervals will be correct, which has a 55.5% chance
historically.
#### 1.3.4.1) sys7: 6, 7, 21, 24, 39, 45, 44
#### 1.3.4.2) sys7:
10, 12, 19, 23, 20, 30, 1
#### 1.3.4.3) sys7: 12, 17, 16, 33, 37, 40, 2
###
1.3.5) The fourth set is a bootstrap sampling without replacement on ALL the
numbers taken from sets ONE (1) to THREE (3).
#### 1.3.5.1) sys7: 13, 19, 28,
30, 41, 42, 44
#### 1.3.5.2) sys7: 3, 10, 12, 18, 22, 26, 41
#### 1.3.5.3) sys7:
6, 14, 16, 28, 31, 33, 44
#### 1.3.5.4) sys7: 3, 5, 26, 28, 34, 38, 40
###
1.3.6) The fifth set is a bootstrap sampling without replacement on ALL the
numbers taken from a Quickpick SIX (6) boards of sys7 at $21.00.
#### 1.3.6.1)
sys7: 1, 13, 14, 24, 25, 35, 37
#### 1.3.6.2) sys7: 14, 17, 18, 25, 29, 31, 38
#### 1.3.6.3) sys7: 1, 8, 12, 18, 29, 38, 42
### 1.3.7) Total sets: FIVE (5);
Total boards: FIFTEEN (15); Total bets ($10.50 EACH): FIVE (5); Total cost:
$52.50.
