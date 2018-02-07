Haugh_03_zcb_fOptions
==================================================================================================================================
## Motivational Buckets
### 1.1) (size: ) In Coursera's Financial Engineering
and Risk Management (FERM) course, Prof. Haugh uses an Excel spreadsheet to
specify and build a binomial tree model in terms of Black-Scholes parameters.
The model can then be used to price European and American options for EITHER an
underlying interest rate OR zero coupon bond. Note that the binomial tree model
price converges to Black-Scholes price as n (the number of periods) approaches
INFINITY (oo).
### 1.2) (size: ) The Black-Scholes parameters can be divided
into TWO (2) categories. The first category consists of parameters that are NOT
actually used in the option pricing, but rather used in the calculation of the
interest rates. This category includes the following parameters: (i) r - the
initial interest rate; (ii) u - up-move rate; (iii) d - down-move rate; (iv) q -
risk-neutral probability of up-move rate (default: 0.5); (v) qInv - inverse
risk-neutral probability of down-move rate (1-q). The second category consists
of parameters that are used in the option pricing. This category includes
parameters: (vi) S - initial underlying security price; (vii) X - the exercise
price; (viii) type - "European" OR "American"; and (ix) flag - "put" OR "call".
### 1.3) (size: 99) The steps to option pricing include: (i) construct a short-
rate interest lattice for R; (ii) construct a zero-coupon bond lattice for R;
and (iii) construct the bond lattice (there is a difference between "forward"
and "futures").
### 1.4) (size: 112) The R package "fOptions" contain several
functions that are useful: (i) CRRBinomialTreeOption - return an object fOptions
that consists of the evaluated option price; (ii) BinomialTreeOption() - return
a matrix that contains the intermediate option prices from period 0 to n; (iii)
BinomialTreePlot() - plot the matrix.

```{.python .input}
%%R
BscdCustomModel <- function(r, u, d, q, n)
{
  #---  Check that arguments are valid
  if( r <= 0  )
    stop("r MUST be greater than ZERO (0)")
  if( u <= 1  )
    stop("u MUST be greater than ONE (1)")
  if( d <= 0  )
    stop("d MUST be greater than ZERO (0)")
  if( d >= 1  )
    stop("d MUST be less than ONE (1)")
  if( q <= 0 )
    stop("q MUST be greater than ZERO (0)")
  if( q >= 1 )
    stop("q MUST be less than ONE (1)")
  qInv  <- 1 - q
  
  #---  Construct an interest rate lattice (base) matrix using initial value of r
  #       (1) matrix n1 x n1, where n1=n+1 because of initial term at n=0
  #       (2)   1         2         3         4
  #           1 r         r*u^1*d^0 r*u^2*d^0 r*u^3*d^0
  #           2           r*u^0*d^1 r*u^1*d^1 r*u^2*d^1
  #           3                     r*u^0*d^2 r*u^1*d^2
  #           4                               r*u^0*d^3
  n1 <- n+1
  basMtx <- matrix( rep(0, n1*n1), nrow=n1, ncol=n1)
  invMtx <- matrix( rep(0, n1*n1), nrow=n1, ncol=n1)
  for( i in 1:nrow(basMtx) )
  {
    for( j in i:ncol(basMtx) )
    {
      pd <- i-1
      pu <- j-1-pd
      basMtx[i,j] <- r*u^pu*d^pd
      if( basMtx[i,j] != 0 )
        invMtx[i,j] <- 1 / (1+basMtx[i,j])
   }
  }
  
  #---  Construct a zero coupon bond (zcb) lattice matrix based on basMtx and final payoff=1
  #       (1) Set last column of zcbMtx to ONE (1)
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
  zcbMtx <- matrix( rep(0, n1*n1), nrow=n1, ncol=n1)
  zcbMtx[, n1] <- rep(1, n1)
  for( j in (n1-1):1 )
  {
    for( i in j:1 )
    {
      ld <- zcbMtx[i+1, j+1]
      lu <- zcbMtx[i, j+1]
      zcbMtx[i,j] <- (q*lu+qInv*ld) / (1+basMtx[i,j])
    }
  }
  
  #---  Return an object of class Bscd, i.e. Black-Scholes Discrete model
  #
  ret.Bscd <- list("r"      = r,
                   "b"      = 0,
                   "n"      = n,
                   "n1"     = n1,
                   "Time"   = n,
                   "sigma"  = NULL,
                   "R"      = NULL,
                   "RInv"   = NULL,
                   "u"      = u,
                   "d"      = d,
                   "q"      = q,
                   "qInv"   = 1 - q,
                   "sRateMtx" = basMtx,
                   "RInvMtx"  = invMtx,
                   "zRateMtx" = zcbMtx
  )
  class(ret.Bscd) <- "Bscd"
  ret.Bscd
}
```

### 1.2.1) (size: ) The first category of the Black-Scholes parameters are used
to calculate the interest rate lattice, using the following parameters: (i) R -
initial interest rate; (ii) u - upmove rate; (iii) d - downmove rate; (iv) q -
risk neutral probability (default: 0.5); and (v) n - number of periods.
###
1.2.2) The function BscdCustomModel() accepts FIVE (5) parameters as above. The
function returns an object of class "Bscd", i.e. Black-Scholes Discrete class,
that consists of several variables, e.g. q, u, d, etc, and THREE (3) matrices:
(i) short-rate interest lattice (sRateMtx); (ii) inverse short-rate interest
lattice (RInvMtx); and (iii) zero-coupon bond lattice (zRateMtx). These matrices
are based on an initial value of ONE (1) and could be used to construct option
binomial tree prices. To obtain the actual prices, just multiply the matrix by
the initial value, e.g. 100*zRateMtx.

```{.python .input}
%%R
R       <- 0.05
up      <- 1.1
dn      <- 0.9
q       <- 0.5
n.step  <- 10
model   <- BscdCustomModel(R, up, dn, q, n.step)
str(model)
model$zRateMtx[1,1]*100
```

```{.python .input}
%%R
BscdPayTwoLeafMtx <- function( q, finalNum, df=1 )
{
  qInv        <- 1-q
  n           <- length(finalNum)
  if( length(df) == 1 )
    df        <- matrix(rep(df, n*n), nrow=n, ncol=n)
  retMtx      <- matrix(rep(0, n*n), nrow=n, ncol=n)
  retMtx[, n] <- finalNum
  for( j in (n-1):1 )
  {
    for( i in j:1 )
    {
      ld <- retMtx[i+1, j+1]
      lu <- retMtx[i,   j+1]
      retMtx[i,j] <- df[i,j] * (q*lu + qInv*ld)
    }
  }
  retMtx
}
BscdPayTwoLeafAMtx <- function( q, finalNum, finalMtx, df=1 )
{
  qInv        <- 1-q
  n           <- length(finalNum)
  if( length(df) == 1 )
    df        <- matrix(rep(df, n*n), nrow=n, ncol=n)
  retMtx      <- matrix(rep(0, n*n), nrow=n, ncol=n)
  retMtx[, n] <- finalNum
  for( j in (n-1):1 )
  {
    for( i in j:1 )
    {
      ld <- retMtx[i+1, j+1]
      lu <- retMtx[i,   j+1]
      retMtx[i,j] <- max( finalMtx[i,j], df[i,j] * (q*lu + qInv*ld) )
    }
  }
  retMtx
}
BscdOptionPrice <- function( model, S, X, subMtx=NULL, n=NULL, 
                             TypeFlag=c("ce", "pe", "ca", "pa") )
{
  #---  Check that arguments are valid
  if( is.null(n) )  
    n <- model$n
  if( n > model$n )
    stop("n CANNOT be greater than model$n")
  if( !is.null(subMtx) )
  {
    if( nrow(subMtx) != ncol(subMtx) )
      stop("subMtx MUST be a square matrix")
    if( n > nrow(subMtx) )
      stop("n CANNOT be greater than nrow(subMtx)")
  }
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
  if( is.null(subMtx) )
    stockMtx <- model$sRateMtx * S
  else
    stockMtx <- subMtx
  
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
  if( is.null(model$RInv) )
    df <- model$RInvMtx
  else
    df <- model$RInv
  difNum    <- stockMtx[, n1]-X
  difNum    <- difNum[1:n1]
  if( ceBln )
  {
    flag      <- 1
    payNum    <- sapply(flag*difNum, max, 0)
    ceMtx     <- BscdPayTwoLeafMtx( model$q, payNum, df )
  }
  if( peBln )
  {
    flag      <- -1
    payNum    <- sapply(flag*difNum, max, 0)
    peMtx     <- BscdPayTwoLeafMtx( model$q, payNum, df )
  }
  difMtx    <- stockMtx-X
  difMtx    <- difMtx[1:n1, 1:n1]
  if( caBln )
  {
    flag      <- 1
    payNum    <- sapply(flag*difNum, max, 0)
    payMtx    <- flag*difMtx[1:n, 1:n]
    caMtx     <- BscdPayTwoLeafAMtx( model$q, payNum, payMtx, df )
  }
  if( paBln )
  {
    flag      <- -1
    payNum    <- sapply(flag*difNum, max, 0)
    payMtx    <- flag*difMtx[1:n, 1:n]
    paMtx     <- BscdPayTwoLeafAMtx( model$q, payNum, payMtx, df )
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

### 1.3.1) (size: ) Each of the following questions should be answered by
building a binomial tree model for the short-rate interest rate, R[i,j]. The
lattice parameters are as follows: (i) r[0,0]=0.05, (ii) u=1.1; (iii) d=0.9;
(iv) q=1/2; and (v) n=10.

```{.python .input}
%%R
S       <- 100
bondMtx <- model$zRateMtx * S
bondMtx
CC      <- 0
dfMtx   <- model$RInvMtx
n       <- 4
n1      <- n+1
dfMtx   <- dfMtx[1:n1, 1:n1]
hBonMtx <- BscdPayTwoLeafMtx( model$q, rep(100, n1), dfMtx )
hBonMtx
difNum  <- bondMtx[, n1]-CC
difNum  <- difNum[1:n1]
flag    <- 1
payNum  <- flag*difNum
fwdMtx  <- BscdPayTwoLeafMtx( model$q, payNum, dfMtx )
fwdMtx
S*fwdMtx[1,1]/hBonMtx[1,1]
futMtx  <- BscdPayTwoLeafMtx( model$q, payNum, 1 )
futMtx
X       <- 80
dfMtx   <- model$RInvMtx
n       <- 6
n1      <- n+1
dfMtx   <- dfMtx[1:n1, 1:n1]
difNum  <- bondMtx[, n1]-X
difNum  <- difNum[1:n1]
difMtx  <- bondMtx-X
difMtx  <- difMtx[1:n1, 1:n1]
flag    <- 1
payNum  <- sapply(flag*difNum, max, 0)
payNum
payMtx  <- flag*difMtx[1:n, 1:n]
payMtx
caMtx   <- BscdPayTwoLeafAMtx( model$q, payNum, payMtx, dfMtx)
caMtx
```

### 1.4.1) (size: ) Compute the price of a zero-coupon bond (ZCB) that matures
at time t=10 and that has face value 100.
### 1.4.2) Compute the price of a
forward contract on the same ZCB of the previous question where the forward
contract matures at time t=4.
### 1.4.3) Compute the initial price of a futures
contract on the same ZCB of the previous two questions. The futures contract has
an expiration of t=4.
### 1.4.4) Compute the price of an American call option on
the same ZCB of the previous three questions. The option has expiration t=6 and
strike =80.

```{.python .input}
%%R
BscdPayTwoLeafBMtx <- function( q, finalNum, finalMtx, df=1 )
{
  qInv        <- 1-q
  n           <- length(finalNum)
  if( length(df) == 1 )
    df        <- matrix(rep(df, n*n), nrow=n, ncol=n)
  retMtx      <- matrix(rep(0, n*n), nrow=n, ncol=n)
  retMtx[, n] <- finalNum
  for( j in (n-1):1 )
  {
    for( i in j:1 )
    {
      ld <- retMtx[i+1, j+1]
      lu <- retMtx[i,   j+1]
      retMtx[i,j] <- df[i,j] * (finalMtx[i,j] + q*lu + qInv*ld)
    }
  }
  retMtx
}
```

```{.python .input}
%%R
CC      <- 0.045
dfMtx   <- model$RInvMtx
n       <- 10
n1      <- n+1
dfMtx   <- dfMtx[1:n1, 1:n1]
difNum  <- (model$sRateMtx[, n1] - CC) * dfMtx[, n1]
difNum  <- difNum[1:n1]
difMtx  <- model$sRateMtx-CC
difMtx  <- difMtx[1:n1, 1:n1]
flag    <- 1
payNum  <- flag*difNum
payNum
payMtx  <- flag*difMtx[1:n, 1:n]
payMtx
swapMtx <- BscdPayTwoLeafBMtx( model$q, payNum, payMtx, dfMtx )
swapMtx
swapMtx[1,1] <- dfMtx[1,1] * (model$q*swapMtx[1,2] + model$qInv*swapMtx[2,2])
1000000*swapMtx[1,1]
X       <- 0.00
n       <- 5
n1      <- n+1
dfMtx   <- model$RInvMtx[1:n1, 1:n1]
difNum  <- swapMtx[, n1]
difNum  <- difNum[1:n1]
flag    <- 1
payNum  <- sapply(flag*difNum, max, 0)
payNum
swap.op <- BscdPayTwoLeafMtx( model$q, payNum, dfMtx )
swap.op
1000000*swap.op[1,1]
```

### 1.4.5) Compute the initial value of a forward-starting swap that begins at
t=1, with maturity t=10 and a fixed rate of 4.5%. (The first payment then takes
place at t=2 and the final payment takes place at t=11 as we are assuming, as
usual, that payments take place in arrears.) You should assume a swap notional
of 1 million and assume that you receive floating and pay fixed. Note: The
example in the lecture for swaps starts payments at t = 1. The problem above
starts payments at t = 2.
### 1.4.6) Compute the initial price of a swaption
that matures at time t=5 and has a strike of 0. The underlying swap is the same
swap as described in the previous question with a notional of 1 million. To be
clear, you should assume that if the swaption is exercised at t=5 then the owner
of the swaption will receive all cash-flows from the underlying swap from times
t=6 to t=11 inclusive. (The swaption strike of 0 should also not be confused
with the fixed rate of 4.5% on the underlying swap.)
