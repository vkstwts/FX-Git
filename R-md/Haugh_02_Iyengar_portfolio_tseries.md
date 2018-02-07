Haugh_02_Iyengar_portfolio_tseries
==================================================================================================================================
## Motivational Buckets
### 1.1) (size: ) In Coursera's Computational Finance
and Financial Econometrics (CFFE) course, Prof. Zivot uses an R function
efficient.frontier() to optimize a portfolio of securities and to specify a set
of linear constraints. However, the model has only TWO (2) linear constraints:
(a) EACH weight MUST be positive; and (b) the sum of weights MUST EQUAL to ONE
(1). However, there is an idiosyncratic risk - weight on ANY security can be
large, i.e. up to ONE (1) - in this model.
### 1.2) (size: ) In a separate
Coursera's Financial Engineering and Risk Management (FERM) course, Prof.
Iyengar uses an Excel spreadsheet to optimize a portfolio of securities and to
specify a set of linear constraints. The model can be used to find the "best"
portfolio and its optimal weights with a set of linear constraints that can
easily be extended to more than TWO (2) constraints. For example, we can add a
constraint that EACH weight must be LESS than TEN (10%) percent. 
### 1.3)
(size: ) In a presentation "R Tools for Portfolio Optimization", Yollin
recommends the function portfolio.optim() in library(tseries) that can be used
to overcome the idiosyncratic risk. The function accepts several parameters: (i)
x: a numeric vector for returns; (ii) pm: a target return; (iii) covmat: a
covariance matrix; (iv) shorts: a boolean for allow shorts (default: FALSE); (v)
rf: a risk free rate (default: 0.0); (vi) reslow: a numeric vector for the
minimum weights (default: NULL); (vi) reshigh: a numeric vector for the maximum
weights (default: NULL)
### 1.4) (size: ) In a separate Coursera's Computational
Investing (CI) course, Prof. Balch uses an algorithm to optimize a portfolio of
securities. The algorithm uses a brute force method to find the "best" model,
i.e. highest Sharpe ratio, hence it is inefficient and slow.

```{.python .input}
%%R
source("C:/Users/denbrige/100 FXOption/103 FXOptionVerBack/080 FX Git/R-source/PlusBullet.R", echo=FALSE)
symChr  <- c("AAPL", "GLD", "GOOG", "XOM")
symZoo  <- BulletGetHistZoo(symChr, "2011-01-01", "2011-12-31")
cerLst  <- BulletCerParameterLst( symZoo )
sym.eff <- efficient.frontier(cerLst$muHat, cerLst$covHat, alpha.min=-1, alpha.max=1, shorts=FALSE)
summary(sym.eff)
plot(sym.eff, plot.assets=T, col="blue", cex=1)
sym.pom <- portfolio.optim(matrix(cerLst$muHat, nrow=1), reslow=rep(0.1,length(cerLst$muHat)), covmat=cerLst$covHat)
str(sym.pom)
simLst    <- BulletSimulateLst(symZoo, sym.pom$pw)
mSd.num   <- simLst$sd
mRet.num  <- simLst$meanRet
points(mSd.num, mRet.num, col="red", pch=16)
text(mSd.num, mRet.num, paste("Sharpe=", round(simLst$sharpe, 1)))
abline(h=mRet.num)
abline(v=mSd.num)
```

### 1.1.1) (size: 16) The R script "PlusBullet" sources Zivot's
"portfolio_noshorts" R script, and adds several functions to analyze and
optimize a portfolio. We want to analyze and compare the results of function
efficient.frontier() in "PlusBullet" R script and Yollin's effFrontier()
function, which utilizes portfolio.optim() function in library(tseries).
###
1.1.2) The first task is to download Yahoo's financial data for EACH security in
our portfolio. The function BulletGetHistZoo() accepts THREE (3) inputs: (a)
symChr: a character vector of symbols; (b) startDate: a string with format
"yyyy-mm-dd" for start date; and (c) finishDate: a string with format "yyyy-mm-
dd" for end date. It returns a zoo object containing the adjusted closing prices
for EACH symbol.
### 1.1.3) The second task is to create a list containing the
Constant Expected Returns parameters. This can be done by passing the zoo
object, obtained from the previous function, to the function
BulletCerParameterLst. This function returns a list that contains FIVE (5)
objects: (i) muHat: a numeric vector for mean of returns; (ii) varHat: a numeric
vector for variance of returns; (iii) sigmaHat: a numeric vector for standard
deviation of returns; (iv) covHat: a covariance matrix; and (v) corHat: a
correlation matrix. 
### 1.1.4) The third task is to plot an efficient frontier
containing ALL the symbols. However, this frontier does NOT have constraints on
EACH idiosyncratic risk, i.e. EACH security can have a minimum weight of ZERO
(0) and a maximum weight of ONE (1). For example, we use the function
portfolio.optim() to find the "best" portfolio with AT LEAST THREE (3)
constraints: (a) EACH weight CANNOT be less than 0.1; (b) the sum of weights
MUST EQUAL to ONE (1); and (c) no shorts allowed. Fortunately, we can use the
values in "CerLst" with only slight modifications: (i) x: we need to convert
cerLst$muHat to a 1xn matrix, i.e. "matrix(cerLst$muHat, nrow=1)"; (ii) reslow:
we need to specify a numeric vector for EACH minimum weight, i.e. "rep(0.1,
length(cerLst$muHat))"; and (iii) covmat: "cerLst$covHat" as there is NO
conversion needed.
