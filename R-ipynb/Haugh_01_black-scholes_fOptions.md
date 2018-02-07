Haugh_01_black-scholes_fOptions
==================================================================================================================================
## Motivational Buckets
### 1.1) (size: 28) In Coursera's Financial Engineering
and Risk Management (FERM) course, Prof. Haugh uses an Excel spreadsheet to
specify and build a binomial tree model in terms of Black-Scholes parameters.
The model can then be used to price European and American options for EITHER an
underlying stock OR futures. Note that the binomial tree model price converges
to Black-Scholes price as n (the number of periods) approaches INFINITY (oo).
### 1.2) (size: 28) The Black-Scholes parameters can be divided into TWO (2)
categories. The first category consists of parameters that are NOT actually used
in the option pricing, but rather used in the calculation of the risk neutral
probability (q) and its related discrete terms. This category includes the
following parameters: (i) r - the continuously compounded interest rate; (ii) d
- dividend yield of the underlying security; (iii) n - number of periods; (iv) T
- time to maturity measured in years, e.g. 0.5 means 6 months; (v) sigma - the
annualized volatility of the underlying security, e.g. 0.3 means 30% volatility
p.a. The second category consists of parameters that are used in the option
pricing, but NOT the risk neutral probability. This category includes
parameters: (vi) S - initial underlying security price; (vii) X - the exercise
price; (viii) type - "European" OR "American"; and (ix) flag - "put" OR "call".
### 1.3) (size: 99) The steps to option pricing include: (i) construct a stock
lattice for the underlying; (ii) (optional) construct a futures lattice for the
underlying; (iii) construct the option lattice (there is a difference between
"European" and "American"); (iv) construct an early exercise lattice (only for
"American").
### 1.4) (size: 112) The R package "fOptions" contain several
functions that are useful: (i) CRRBinomialTreeOption - return an object fOptions
that consists of the evaluated option price; (ii) BinomialTreeOption() - return
a matrix that contains the intermediate option prices from period 0 to n; (iii)
BinomialTreePlot() - plot the matrix.

```{.python .input}
%%R
BscdCreateModel <- function(r, b, n, Time, sigma)
{
  #---  Calculate the risk neutral probabilities (q) and (q-1)
  #       (1) Calculate EACH discrete term used by q
  #       (2) R = exp(r*T/n)
  #       (3) RInv = exp(-r*T/n)
  #       (4) Rb = exp((r-b)*T/n)
  #       (5) u = exp(sigma*sqrt(T/n))
  #       (6) d = 1/u
  #       (7) q = ( R - d )   / ( u - d),   if b = 0
  #           q = ( Rb - d )  / ( u - d ),  if b > 0
  #       (8) qInv = 1 - q
  R     <- exp(r*Time/n)
  RInv  <- exp(-r*Time/n)
  Rb    <- exp((r-b)*Time/n)
  u     <- exp(sigma*sqrt(Time/n))
  d     <- 1/u
  if( b==0 )
    q   <- (R-d)/(u-d)
  if( b>0 )
    q   <- (Rb-d)/(u-d)
  qInv  <- 1 - q
  
  #---  Construct a stock lattice (base) matrix using initial value of ONE (1)
  #       (1) matrix n1 x n1, where n1=n+1 because of initial term at n=0
  #       (2)   1         2         3         4
  #           1 u^0*d^0   u^1*d^0   u^2*d^0   u^3*d^0
  #           2           u^0*d^1   u^1*d^1   u^2*d^1
  #           3                     u^0*d^2   u^1*d^2
  #           4                               u^0*d^3
  n1 <- n+1
  basMtx <- matrix( rep(0, n1*n1), nrow=n1, ncol=n1)
  for( i in 1:nrow(basMtx) )
  {
    for( j in i:ncol(basMtx) )
    {
      pd <- i-1
      pu <- j-1-pd
      basMtx[i,j] <- u^pu*d^pd
    }
  }
  
  #---  Construct a futures lattice matrix based on the stock's final payoff at n
  #       (1) Copy last column of basMtx to last column of futMtx
  #       (2) Calculate backwards, starting from the previous column n-1 to 0
  #           using the expected TWO (2) leaf values weighted on risk neutral 
  #           probabilities (q) and (1-q)
  #       (3)   1         2         3         4
  #           1 q*[1,2]+  q*[1,3]+  q*[1,4]+  u^3*d^0
  #             qI*[2,2]  qI*[2.3]  qI*[2,4]  
  #           2           q*[2,3]+  q*[2,4]+  u^2*d^1
  #                       qI*[3,3]  qI*[3,4]
  #           3                     q*[3,4]+  u^1*d^2
  #                                 qI*[4,4]
  #           4                               u^0*d^3
  futMtx <- matrix( rep(0, n1*n1), nrow=n1, ncol=n1)
  futMtx[, n1] <- basMtx[, n1]
  for( j in (n1-1):1 )
  {
    for( i in j:1 )
    {
      ld <- futMtx[i+1, j+1]
      lu <- futMtx[i, j+1]
      futMtx[i,j] <- q*lu+qInv*ld
    }
  }
  
  #---  Return an object of class Bscd, i.e. Black-Scholes Discrete model
  #
  ret.Bscd <- list("r"      = r,
                   "b"      = b,
                   "n"      = n,
                   "n1"     = n1,
                   "Time"   = Time,
                   "sigma"  = sigma,
                   "R"      = R,
                   "RInv"   = RInv,
                   "Rb"     = Rb,
                   "u"      = u,
                   "d"      = d,
                   "q"      = q,
                   "qInv"   = 1 - q,
                   "sRateMtx" = basMtx,
                   "fRateMtx" = futMtx,
                   "fRateNum" = futMtx[1,1]
                   )
  class(ret.Bscd) <- "Bscd"
  ret.Bscd
}
```

### 1.2.1) (size: 28) The first category of the Black-Scholes parameters are
used to calculate the risk neutral probabilities (q), and its related discrete
terms. The formula for risk neutral probability expressed in discrete terms is q
= (R - c - d) / (u - d), where the discrete terms are defined as follows: (i) R
- the discrete interest rate; (ii) c - the annualized dividend yield (also known
as cost-of-carry rate), e.g. 0.1 means 10% p.a.; (iii) d - the downmove rate;
(iv) u - the upmove rate.
### 1.2.2) The discrete terms can be substituted with
the continuous terms, i.e. Black-Scholes parameters, as follows: (i) R =
exp(r*T/n), if c=0; (ii) R - c = exp((r-c)*T/n), if c>0; (iii) u =
exp(sigma*sqrt(T/n)); and (iv) d = 1/u. Therefore, we can rewrite the formula
(q) in continuous terms as q = ( exp((r-c)*T/n) - d ) / ( exp(sigma*sqrt(T/n)) -
d ), where d = 1/u. 
### 1.2.3) The function BscdCreateModel() accepts FIVE (5)
parameters as follows: (i) r - the continuously compounded interest rate; (ii) b
- the annualized dividend yield (OR cost-of-carry rate) of the underlying
security; (iii) n - the number of periods; (iv) Time - time to maturity measured
in years, e.g. 0.5 means 6 months; (v) sigma - the annualized volatility of the
underlying security, e.g. 0.3 means 30% volatility p.a. The function returns an
object of class "Bscd", i.e. Black-Scholes Discrete class, that consists of
several variables, e.g. q, u, d, etc, and TWO (2) matrices: (a) stock price
rates lattice; and (b) futures price rates lattice. These matrices are based on
an initial value of ONE (1) and could be used to construct option binomial tree
prices. To obtain the actual prices, just multiply the matrix by the initial
value S, e.g. 100*sRateMtx.

```{.python .input}
%%R
test.bscd <- BscdCreateModel(0.02, 0.01, 15, 0.25, 0.3)
str(test.bscd)
stock <- 100*test.bscd$sRateMtx
head(stock)
```

```{.python .input}
%%R
append.lst <- function( lst, obj, objStr )
{
  for( i in 1:length(lst) )
  {
    if( is.null(lst[[i]]) )
    {
      lst[[i]] <- obj
      return( lst )
    }
  }
  lst
}
BscdPayTwoLeafMtx <- function( q, finalNum, scalar=1 )
{
  qInv        <- 1-q
  n           <- length(finalNum)
  retMtx      <- matrix(rep(0, n*n), nrow=n, ncol=n)
  retMtx[, n] <- finalNum
  for( j in (n-1):1 )
  {
    for( i in j:1 )
    {
      ld <- retMtx[i+1, j+1]
      lu <- retMtx[i,   j+1]
      retMtx[i,j] <- scalar * (q*lu + qInv*ld)
    }
  }
  retMtx
}
BscdPayTwoLeafAMtx <- function( q, finalMtx, scalar=1 )
{
  qInv        <- 1-q
  n           <- nrow(finalMtx)
  retMtx      <- matrix(rep(0, n*n), nrow=n, ncol=n)
  retMtx[, n] <- finalMtx[ ,n]
  for( j in (n-1):1 )
  {
    for( i in j:1 )
    {
      ld <- retMtx[i+1, j+1]
      lu <- retMtx[i,   j+1]
      retMtx[i,j] <- max( finalMtx[i,j], scalar * (q*lu + qInv*ld) )
    }
  }
  retMtx
}
BscdOptionPrice <- function( model, S, X, n=NULL, TypeFlag=c("ce", "pe", "ca", "pa") )
{
  #---  Check that arguments are valid
  if( is.null(n) )  
    n <- model$n
  if( n > model$n )
    stop("n CANNOT be greater than model$n")
  typeStr <- c("ce", "pe", "ca", "pa")
  if( length(TypeFlag) == 0 )
    stop("TypeFlag MUST be ONE (1) OR MORE of: ce, pe, ca, pa")
  for( i in 1:length(TypeFlag) )
  {
    if( length(which(typeStr==TypeFlag[i])) == 0 )
      stop("TypeFlag MUST be ONE (1) OR MORE of: ce, pe, ca, pa")
  }
  ceBln <- length(which(typeStr[1]==TypeFlag)) > 0
  peBln <- length(which(typeStr[2]==TypeFlag)) > 0
  caBln <- length(which(typeStr[3]==TypeFlag)) > 0
  paBln <- length(which(typeStr[4]==TypeFlag)) > 0
  
  #---  Construct a stock lattice using model and initial price S
  stockMtx <- model$sRateMtx * S
  
  #---  Construct EACH option lattice
  #       (1) matrix n1 x n1, where n1=n+1 because of initial term at n=0
  #       (2) Calculate the LAST column of typeMtx, where type: ce, pe, ca, pa
  #           using the function max() with arguments ZERO (0) and the difference
  #           in strike price (X) and the corresponding nth column stockMtx price.
  #           For a put option, we need to invert the result of max, i.e. -result.
  #       (3) Calculate backwards, starting from the previous column n-1 to 0
  #           using the expected TWO (2) leaf values weighted on risk neutral 
  #           probabilities (q) and (1-q) scaled by 1/R
  #       (4)   1         2         3         n
  #           1 q*[1,2]+  q*[1,3]+  q*[1,4]+  max((flag*stockMtx[1,n]-X),0)
  #             qI*[2,2]  qI*[2.3]  qI*[2,4]  
  #           2           q*[2,3]+  q*[2,4]+  max((flag*stockMtx[2,n]-X),0)
  #                       qI*[3,3]  qI*[3,4]
  #           3                     q*[3,4]+  max((flag*stockMtx[3,n]-X),0)
  #                                 qI*[4,4]
  #           4                               max((flag*stockMtx[4,n]-X),0)
  n1 <- n+1
  difNum    <- S*model$sRateMtx[, model$n1]-X
  if( ceBln )
  {
    flag      <- 1
    payNum    <- sapply(flag*difNum, max, 0)
    ceMtx     <- BscdPayTwoLeafMtx( model$q, payNum, scalar=model$RInv )
  }
  if( peBln )
  {
    flag      <- -1
    payNum    <- sapply(flag*difNum, max, 0)
    peMtx     <- BscdPayTwoLeafMtx( model$q, payNum, scalar=model$RInv )
  }
  difMtx    <- S*model$sRateMtx-X
  if( caBln )
  {
    flag      <-  1
    payMtx    <- matrix( sapply(flag*difMtx, max, 0), nrow=n1, ncol=n1 )
    caMtx     <- BscdPayTwoLeafAMtx( model$q, payMtx, scalar=model$RInv )
  }
  if( paBln )
  {
    flag      <- -1
    payMtx    <- matrix( sapply(flag*difMtx, max, 0), nrow=n1, ncol=n1 )
    paMtx     <- BscdPayTwoLeafAMtx( model$q, payMtx, scalar=model$RInv )
  }
  retBln <- c( ceBln, peBln, caBln, paBln )
  ret.lst <- vector("list", sum(retBln))
  if( retBln[1] & exists("ceMtx") ) ret.lst <- append.lst( ret.lst, ceMtx )
  if( retBln[2] & exists("peMtx") ) ret.lst <- append.lst( ret.lst, peMtx )
  if( retBln[3] & exists("caMtx") ) ret.lst <- append.lst( ret.lst, caMtx )
  if( retBln[4] & exists("paMtx") ) ret.lst <- append.lst( ret.lst, paMtx )
  names(ret.lst) <- typeStr[which(retBln)]
  ret.lst
}
```

### 1.3.1) (size: 99) The function BscdOptionPrice() accepts FIVE (5) parameters
as follows: (i) model - the Black-Scholes Discrete model; (ii) S - the initial
price of the underlying security; (iii) X - the strike price; (iv) n - the
number of periods for the option (default: NULL means the same number of periods
as underlying security); (v) TypeFlag - a character vector for ONE (1) or MORE
option types: (1) "ce": European call; (2) "pe": European put; (3) "ca":
American call; and (4) "pa": American put. The function returns a list
containing the specified number of option binomial tree prices as matrices. The
option price is the value in FIRST row and FIRST column of EACH matrix.

```{.python .input}
%%R
test.op <- BscdOptionPrice(test.bscd, 100, 100)
str(test.op)
chooser.op    <- BscdOptionPrice(test.bscd, 100, 100)
payChooserNum <- apply(cbind(chooser.op$ce[,11], chooser.op$pe[,11]), 1, max, 0) 
payChooserNum <- payChooserNum[1:11]
chooserMtx    <- BscdPayTwoLeafMtx( test.bscd$q, payChooserNum, scalar=test.bscd$RInv )
str(chooserMtx)
```

```{.python .input}
%%R
suppressPackageStartupMessages(require(fOptions))
suppressPackageStartupMessages(require(testthat))
option.ce <- BinomialTreeOption("ce", 100, 100, Time=test.bscd$Time, r=test.bscd$r, b=test.bscd$b, sigma=test.bscd$sigma, n=test.bscd$n)
option.pe <- BinomialTreeOption("pe", 100, 100, Time=test.bscd$Time, r=test.bscd$r, b=test.bscd$b, sigma=test.bscd$sigma, n=test.bscd$n)
option.ca <- BinomialTreeOption("ca", 100, 100, Time=test.bscd$Time, r=test.bscd$r, b=test.bscd$b, sigma=test.bscd$sigma, n=test.bscd$n)
option.pa <- BinomialTreeOption("pa", 100, 100, Time=test.bscd$Time, r=test.bscd$r, b=test.bscd$b, sigma=test.bscd$sigma, n=test.bscd$n)
expect_that( test.op$ce, is_equivalent_to(option.ce) )
expect_that( test.op$pe, is_equivalent_to(option.pe) )
expect_that( test.op$ca, is_equivalent_to(option.ca) )
expect_that( test.op$pa, is_equivalent_to(option.pa) )
par(mfrow=c(1,2))
BinomialTreePlot(test.op$ce)
BinomialTreePlot(test.op$pe)
```

### 1.4.1) (size: 112) The R package "fOptions" contain several functions that
could replicate our results. The function BinomialTreeOption() returns a matrix
that contains the binomial tree option price, which we could use to compare to
our results. It appears that ALL our options prices are correct. The function
BinomialTreePlot() plots an option binomial tree price matrix using a "tree"
graph.
