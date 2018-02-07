Kang_01_real_options_Bscd
==================================================================================================================================
## Motivational Buckets
### 1) (size: 37) Jihun Kang's (2004) thesis titled
"Valuing Flexibilities in Large-Scale Real Estate Development Projects" aims to
develop a set of strategic tools for real estate development projects. TWO (2)
methods are introduced to deal with the inadequecies of the Discounted Cash Flow
(DCF) method: (a) Decision Tree Analysis (DTA); and (b) Real Options Analysis
(ROA). This R Markdown file focused only on the latter method.
### 1.1) (size:
37) Most real estate development (RED), like Research and Development (RND)
projects, do NOT make sense from the Net Present Value (NPV) standpoint. The
most value of RED, like RND, is created when it becomes successful and opens up
new market for a company. In other words, RED, like RND, is a right to acquire
an exclusive and profitable market for the company at a cost incurred during the
development, or research, process. Copeland and Weiner (1990) stated that the
RED, like RND, situation is similar to "paying to see the next card" that
provides the option to move forwards by investing more OR abandoning the
project. Leslie and Michaels (1997) identified SIX (6) variables that can
strategically improve the option value in a RED, or RND, project: (i) S - HIGHER
underlying security price would give HIGHER option value. Like RND, it is
difficult to observe the market price of an asset that does NOT produce an
income stream. We could price S using TWO (2) methods: (a) cost less taxes; OR
(b) DCF of expected net income stream. However, due to the uncertainty of the
market we should not price S based on a random capital gain; (ii) X - as the
strike price INCREASES, the value of a call option DECREASES. The strike price
is equivalent to the present value (PV) of all the fixed costs expected over the
lifetime of the development, or research, process; (iii) sigma - HIGHER
volatility INCREASES the value of the option. This is a critical difference from
the NPV approach. For highly risky projects, options strategy can dramatically
improve their values. We could estimate sigma using TWO (2) approaches: (a) use
a sigma constant for ALL REDs; OR (b) use a sigma based on country risk; (iv) T
- as time to expiration INCREASES so does the value of an option; (v) c -
dividends are considered as cost incurred to keep an option afloat. In RED, we
used the loan interest rate as this is the financial cost of keeping the option
afloat; and (vi) r - an expected INCREASE in the risk free rate RAISES option
value. We could estimate risk free rate using TWO (2) methods: (a) opportunity
cost for developing the real estate; OR (b) interest rate earned from a fixed
deposit.

```{.python .input}
%%R
if( Sys.info()["sysname"] == "Linux" )
  source("~/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE)
if( Sys.info()["sysname"] == "Windows" )
  source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE)
suppressPackageStartupMessages(source( paste0(RegGetRSourceDir(),"PlusBscd.R") ))
suppressPackageStartupMessages(require( stats ))
sigma       <- 0.2
top.month   <- 60
top.year    <- 5
rf          <- 0.03
aqua.bscd   <- BscdCreateModel(rf, 0, top.year, top.year, sigma)
bench.bscd  <- BscdCreateModel(rf, 0.09, top.year, top.year, sigma)
sell.gross  <- 6500/32.35
sell.net    <- sell.gross * 0.94
npv           <- function(ir, cf, t=seq(along=cf)) sum(cf/(1+ir)^t) 
irr           <- function(cf) { uniroot(npv, c(0,1), cf=cf)$root } 
aquaSpotNum   <- function(x) { x * 0.75 }
aquaEqualNum  <- function(x, ir, month) 
{ 
  equalNum  <- (x * 0.88)/month 
  equalVtr  <- rep(equalNum, month)
  npv(ir/12, equalVtr)
}
aquaDeferNum  <- function(x, ir, month)
{
  equalNum      <- (x * 0.4)/month
  equalVtr      <- rep(equalNum, month)
  equalVtr[month]  <- equalVtr[month] + (x * 0.6)
  npv(ir/12, equalVtr)
}
benchNum      <- function(x) { x * 0.75 }
sell.gross
aquaSpotNum(sell.gross)
aquaEqualNum(sell.gross, rf, top.month)
aquaDeferNum(sell.gross, rf, top.month)
benchNum(sell.gross)
aquaSpot.op   <- BscdOptionPrice( aqua.bscd, sell.net, aquaSpotNum(sell.gross) )
aquaEqual.op  <- BscdOptionPrice( aqua.bscd, sell.net, aquaEqualNum(sell.gross, rf, top.month) )
aquaDefer.op  <- BscdOptionPrice( aqua.bscd, sell.net, aquaDeferNum(sell.gross, rf, top.month) )
bench.op      <- BscdOptionPrice( bench.bscd, sell.net, benchNum(sell.gross) )
obs.op <- c(aquaSpot.op$ce[1,1],
            aquaEqual.op$ce[1,1],
            aquaDefer.op$ce[1,1],
            bench.op$ce[1,1])
```

### 1.1.1) (size: 37) Century Property provides THREE (3) financing options: (a)
spot payment at 25% discount; (b) equal payment at 12% discount (up to 14%) over
the lifetime of the development; and (c) 40-60 payment at 0% discount, where 40%
of the purchase price is scheduled as equal payments over the lifetime of the
development, and 60% of the purchase price is at the completion of the
development (TOP). Depending on which option you choose, this will result in
different fixed costs, i.e. X. Also, we used (d) a benchmark of borrowing the
total amount of the fixed cost at c interest rate with spot payment at 25%
discount. 
### 1.1.2) We valued the FOUR (4) options above while keeping the
remaining FOUR (4) parameters of the Black-Scholes Discrete model constant: (i)
S - selling price less 6% CGT, where selling price is assumed to be the listed
purchase price; (ii) sigma - 0.2; (iii) T - 68/12; and (iv) r - 0.03;

```{.python .input}
%%R
plot(seq(1:4), obs.op, col=seq(1:4),
     xlab="", ylab="Real Option Value (ROV in SGD 1K)", main="Comparison of Financing Options",
     pch=19)
legend(3.5, max(obs.op), c("Spot","Equal","Defer","Bench"), col=seq(1:4),
       pch=19)
text(seq(1,4,by=1)+0.1, obs.op, round(obs.op,1), col=seq(1:4))

BscdSaMtx <- function(model, S, X, row.seq=seq(0.01, 0.2, by=0.01), col.seq=seq(1.0, 0.7, by=-0.05))
{
  sMtx <- matrix(0, nrow=length(row.seq), ncol=length(col.seq))
  for( vRow in seq_along(row.seq) )
  {
    sa.bscd <- BscdCreateModel(model$r, model$b, model$n, model$Time, row.seq[vRow])
    for( wCol in seq_along(col.seq) )
    {
      sa.op           <- BscdOptionPrice( sa.bscd, S*col.seq[wCol], X, TypeFlag="ce" )
      sMtx[vRow,wCol] <- sa.op$ce[1,1]
    }
  }
  sMtx
}
sMtx <- BscdSaMtx(aqua.bscd, S=sell.net, X=aquaEqualNum(sell.gross, rf, top.month))
round(sMtx,1)
```

### 1.1.3) The estimated sigma (0.2) is, to a large extent, a subjective
estimate and thus it warrants a sensitivity analysis about this value. We show
the result of option value sensitivity in relation to volatility (sigma) and
asset value (S). For the SECOND option - equal payment - as asset value (net
selling price) DECREASES from left (100%) to right (70%) column, the option
price DECREASES. However, as volatility increases from top (1%) to bottom (20%)
row, the option price INCREASES. The option price is positive for ALL
parameters, except when asset value is LESS THAN EIGHTY FIVE (85%) and
volatility is LESS THAN FIVE (5%) percent.
