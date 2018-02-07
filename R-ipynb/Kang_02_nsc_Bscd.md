Kang_02_nsc_Bscd
==================================================================================================================================
## Motivational Buckets
### 1) (size: 33) Jihun Kang's (2004) thesis titled
"Valuing Flexibilities in Large-Scale Real Estate Development Projects" aims to
develop a set of strategic tools for real estate development projects. TWO (2)
methods are introduced to deal with the inadequecies of the Discounted Cash Flow
(DCF) method: (a) Decision Tree Analysis (DTA); and (b) Real Options Analysis
(ROA). This R Markdown file focused only on the latter method.
### 1.1) (size:
33) Most real estate development (RED), like Research and Development (RND)
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
suppressPackageStartupMessages(require( fOptions ))
suppressPackageStartupMessages(require( testthat ))
sigma       <- 0.25
top.step    <- 12
top.year    <- 12
rf          <- 0.055
cc          <- 0.06
nsc.bscd    <- BscdCreateModel(rf, cc, top.step, top.year, sigma)
p.op        <- BscdOptionPrice(nsc.bscd, S=13562.513, X=2023.163, n=12, TypeFlag="ca")
p5.op       <- BscdOptionPrice(nsc.bscd, S=2361.567, X=3130.788, subMtx=p.op$ca, n=10, TypeFlag="ca")
p4.op       <- BscdOptionPrice(nsc.bscd, S=4274.987, X=4552.729, subMtx=p5.op$ca, n=8, TypeFlag="ca")
p3.op       <- BscdOptionPrice(nsc.bscd, S=1174.826, X=1308.519, subMtx=p4.op$ca, n=6, TypeFlag="ca")
p2.op       <- BscdOptionPrice(nsc.bscd, S=3260.798, X=2795.515, subMtx=p3.op$ca, n=4, TypeFlag="ca")
p1.op       <- BscdOptionPrice(nsc.bscd, S=1373.314, X=1203.098, subMtx=p2.op$ca, n=2, TypeFlag="ca")
```

### 1.1.1) (size: 19) The example above is taken from Kang's (2004) case study
on "Valuing Flexibilities in the development of New Songdo City (NSC)". We
reproduce the SIX(6)-stage sequential compound option for valuing NSC to
demonstrate the use of Real Options Analysis (ROA) as a strategic tool. The FIVE
(5) parameters used for the model and its assumptions are: (i) sigma=0.25 -
HIGHER than the range for the US real estate market, and HIGHER than the
volatility of the Korean housing index and Seoul's office survey as the
properties in the entirely new city are likely to be more volatile in the
beginning than the properties in mature cities; (ii) rf=0.055 - the SAME as rate
of TEN(10)-year, the longest term, Korean government bond, since it matches
quite well the duration of the project; (iii) c=0.06 - weighted AVERAGE of cap
rates for ALL the properties in the project; (iv) T=12 - expected years to
completion; (v) n=12 - the construction company would have an option to wait at
least TWO (2) years before starting the next phase, therefore n=12,10,8... for
Phase=6,5,4...
### 1.1.2) The FIRST option is for Phase SIX (P6) of the project:
(a) S=1,117,022 - PV of benefits for P6; (b) X=2,023,163 - PV of costs for P6.
The SECOND option is for Phase FIVE (P5) of the project: (a) S=2,361,567; (b)
X=3,130,788. The THIRD option is for Phase FOUR (P4) of the project: (a)
S=4,274,987; (b) X=4,552,729. The FOURTH option is for Phase THREE (P3) of the
project: (a) S=1,174,826; (b) X=1,308,519. The FIFTH option is for Phase TWO
(P2) of the project: (a) S=3,260,798; (b) X=2,795,515. Finally, the SIXTH option
is for Phase ONE (P1) of the project: (a) S=1,373,314; (b) X=1,203,098. Note:
ALL the options have TypeFlag="ca" and the PVs of benefits and costs are in US$
1K. You need to sum the PVs of benefits and costs for EACH Phase in separate
columns from Figure 6.10 (page 111). The net value created with the project is
1,113,928 (in thousands US$), which is the option price for P1 (1,370,478) LESS
the PV of land cost for P1 (256,550) that was committed to purchase this option.

```{.python .input}
%%R
str(nsc.bscd)
BinomialTreePlot(p.op$ca)
BinomialTreePlot(p5.op$ca)
BinomialTreePlot(p4.op$ca)
BinomialTreePlot(p3.op$ca)
BinomialTreePlot(p2.op$ca)
BinomialTreePlot(p1.op$ca)
op.ca <- BinomialTreeOption("ca", S=13562.513, X=2023.163, Time=nsc.bscd$Time, r=nsc.bscd$r, b=nsc.bscd$b, sigma=nsc.bscd$sigma, n=nsc.bscd$n)
BinomialTreePlot(p.op$ca)
BinomialTreePlot(op.ca)
obs.op <- c(p.op$ca[1,1],
            p5.op$ca[1,1],
            p4.op$ca[1,1],
            p3.op$ca[1,1],
            p2.op$ca[1,1],
            p1.op$ca[1,1])
plot(seq(1,6,by=1), obs.op, col=seq(1:6),
     xlab="", ylab="Real Options Value (ROV in USD 1K)", main="Comparison of Phase SIX (6) to ONE (1) Options",
     pch=19)
legend(5.5, max(obs.op), c("P6","P5","P4","P3","P2","P1"), col=seq(1:6),
       pch=19)
text(seq(1,6,by=1)+0.1, obs.op, round(obs.op,1), col=seq(1:6))
```

### 1.1.3) (size: 33) The estimated sigma (0.25) is, to a large extent, a
subjective estimate and thus it warrants a sensitivity analysis about this
value. We show the result of option value sensitivity in relation to volatility
(sigma) and asset value (S).
