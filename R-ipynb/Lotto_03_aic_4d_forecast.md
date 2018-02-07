Lotto_03_aic_4d_forecast
==================================================================================================================================
## Motivational Buckets
### 1) (size: 30) This study is NOT just about the model
accuracy and prediction, BUT also the selection process for new estimates. We
MUST identify ANY deficiency in the current models, compare them with new
prospective models, and understand WHY one model is "better" than another. Also,
preference is given to the application, analysis and synthesis that are
difficult to perform using the Shiny package.
### 1.1) I do NOT quite understand
why a diff.arima model is "better" at predicting than a log.diff.arima model. By
"better", I mean TWO (2) things: (a) confidence intervals are tighter; (b)
actual results lie within confidence intervals.
### 1.2) Historically, some
plots would make the data more interesting: (i) the actual number vs confidence
interval should be plotted for past number of draws.
### 1.3) Also, we MUST size
up, weigh and value EACH number in an estimate. We should determine if the size,
weight OR value of a number has changed with respective to predicted estimates
and actual results.

```{.python .input}
%%R
source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusLotto.R", echo=FALSE)
lotto.str <- "4d"
user.str <- ask("Update ALL Lotto results (y OR <blank> for n)?")
if( user.str=="y" ) LottoUpdateNum()
10000/lottoArimaNum(lotto.str, 2, FALSE)
10000/lottoArimaNum(lotto.str, 2, TRUE)
LottoArimaConf(lotto.str, 2)
LottoResult(lotto.str)
```

### 1.1.1) (size: 8) We must ensure that the files contain the latest results.
First, we check the predicted confidence intervals vs the actual results.
###
1.1.2) The 80% confidence interval means that if a sample were taken, then the
probability of a true value lying within that confidence interval is 80%. In
other words, there is a 20% chance that the confidence interval is off the mark.
Coincidentally, for toto Draw No. 2824, EVERY number (including SUM) were within
their respective 80% confidence intervals.

```{.python .input}
%%R
rawDfr <- fileReadDfr(lotto.str)
conf.list <- list()
for( startNum in 1:10 )
{
  conf.list[[startNum]] <- LottoArimaConf(lotto.str, startNum, TRUE)
}
par(mar=c(4,4,0,2), mfrow=c(6,2))
for( rankNum in 1:3 )
{
  for( rollNum in 1:4 )
  {
    plot(1:10,type="n",xlim=c(0,9),ylim=c(1,10), xlab="Confidence Interval",ylab="StartNum")
    for( startNum in 1:10 )
    {
      confDfr <- conf.list[[startNum]]
      lowerNum <- confDfr[rollNum, 1]
      upperNum <- confDfr[rollNum, 2]
      segments(lowerNum, startNum, upperNum, startNum, col='grey')
      if( startNum > 1 )
      {
        resultChr <- rawDfr[ startNum-1, rankNum+2]
        resultNum <- as.numeric(substr(resultChr, rollNum, rollNum))
        if( resultNum >= lowerNum & resultNum <= upperNum )
        {
          points(resultNum, startNum, col='grey')
        }
        else
        {
          if( rankNum == 1 ) points(resultNum, startNum, col='red', pch=19)
          if( rankNum == 2 ) points(resultNum, startNum, col='magenta', pch=19)
          if( rankNum == 3 ) points(resultNum, startNum, col='blue', pch=19)
        }
      }
    }
  }
}
```

### 1.2.1 (size: 22) Historically, there were TWELVE (12) incorrect confidence
intervals in the last NINE (9) draws. This works out to 12/63 (or 19.0%) error
rate, which is marginally below the 20% CI error rate. Of which, ONE (1) draw
had ZERO (0) error, FIVE (5) draws had ONE (1) error, TWO (2) draws had TWO (2)
errors, and ONE (1) draw had THREE (3) errors. This means that the chance of at
least ONE (1) error in a draw is 88.9%; at least TWO (2) errors in a draw is
33.3%; and at least THREE (3) errors in a draw is 11.1%. [Note: The actual
result had THREE (3) errors, which is a probability of 11.1%, compared with our
assumption of BETWEEN ONE (1) and TWO (2) errors.]
### 1.2.2 If we look at
individual digits in the last NINE (9) draws, digit #1 had ONE (1) error
(11.1%), digit #2 had THREE (3) errors (33.3%); digit #3 had SEVEN (7) errors
(77.8%); and digit #4 had ONE (1) error (11.1%).
### 1.2.3 Next, we rank the
digits based on eye-balling the errors and intervals, from most likely to be
correct as the highest: digit #4, #digit #2, digit #1, digit #3. Hence, this
results in the confidence intervals being ranked as follows:

####  lower upper
#### 4     2     9
#### 2     1     9
#### 1     0     2
#### 3     0     6

###
1.2.4 The first possibility is digits ONE (1) to FOUR (4) will be <=2 unlikely,
0 unlikely, <=6 highly unlikely, <=2 highly unlikely, respectively. This assumes
that at least TWO (2) intervals will be incorrect, which has a 33.3% chance
historically.
#### 1.2.4.1 ibet: 5877, 4296, 4176, 6196, 3186
### 1.2.5 A trade
off would be that digits ONE (1) to FOUR (4) will be <=2 likely, 0 unlikely, <=6
highly unlikely, <=2 highly unlikely, respectively. This assumes that ONLY ONE
(1) interval will be incorrect, which has a 88.9% chance.
#### 1.2.5.1 ordinary:
1578, 1587, 1476, 2197, 1297, 1386
