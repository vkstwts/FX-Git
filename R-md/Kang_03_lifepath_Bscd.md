Kang_03_lifepath_Bscd
==================================================================================================================================
## Motivational Buckets
### 1) (size: 26) An epiphany on why different people
with different life paths (LPs) have varying success with marriage, career and
relationships.
### 1.1) (size: 14) An LP does NOT depend on established
relationships, but on ongoing variable dynamics. ONE (1) variable is sincerity,
and this value can be observed using communication skills: (i) body language;
(ii) tone of voice; and (iii) facial expression. The HIGHER the observed value
of EACH skill, the LOWER the sincerity.
### 1.2) (size: 26) We construct a
binomial tree for EACH relationship, using the risk-neutral probabilities and
final payoffs approach, such that the price (value at root) represents the value
of EACH relationship. A relationship is analogous to a stock, where it may
perform EITHER an up-move OR down-move at EACH node. For example, an up-move
could happen after a person joint ventures with a partner to invest in a
property. Conversely, a down-move could happen because a person has been cheated
by a partner to invest in a scam. NOT EVERY move MUST involve a tangible asset,
e.g. a partner cheated by lying is a down-move.
### 1.3) (size: 26) Black-
Scholes adjusted parameters: (i) rf - the HIGHER the rf rate, the HIGHER the
relationship value; (ii) c - the HIGHER the carry cost, the LOWER the
relationship value; (iii) sigma - the HIGHER the volatility, the HIGHER the
relationship value; (iv) n - the HIGHER the number of periods, the HIGHER the
"perceived" relationship, BUT it does NOT alter the outcome of the value; (v) T
- time to maturity in years.
### 1.4) (size: 26) For example, if we want to
invest in a joint property with a sibling: (i) rf=0.02 - this could be a
"perceived" probability of success, based on biases, etc, BUT it has a indirect
relationship to success via the risk-neutral probabilities; (ii) c=0.03 - this
could be a maintenance factor, where HIGHER insincerity implies HIGHER
maintenance cost; (iii) sigma=0.2 - this could be a build factor, where the
HIGHER the build factor, the HIGHER the relationship value; (iv) n=4 - the
period has to be LARGER for a HIGHER "perceived" relationship, but it does NOT
produce a HIGHER relationship value; (v) T=1 - the shorter the time to maturity,
the LESS "inherent" risk associated with family.

```{.python .input}
%%R
if( Sys.info()["sysname"] == "Linux" )
  source("~/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE)
if( Sys.info()["sysname"] == "Windows" )
  source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE)
suppressPackageStartupMessages(source( paste0(RegGetRSourceDir(),"PlusBscd.R") ))
suppressPackageStartupMessages(require( fOptions ))
suppressPackageStartupMessages(require( testthat ))
rf          <- 0.01
cc          <- 0.02
sigma       <- 0.25
top.step    <- 5
top.year    <- 5
filial.bscd <- BscdCreateModel(rf, cc, top.step, top.year, sigma)
filial.op   <- BscdOptionPrice(filial.bscd, S=100, X=100)
```

### 1.1.1) (size: 14) For example, the FIRST model is for a relationship between
daughter/son-in-law and mother. We want to co-invest in a property in the
Philippines and we want to calculate the relationship value from this joint
venture: (i) rf=0.01 - this could be a "perceived" probability of success and
offset factor, where it has to offset the T parameter by lowering its value
(default: 0.03); (ii) c=0.02 - this could be a maintenance factor, where it has
to be LOWER than the upper value for a hypothetical insincere/insincere (c:
0.06=0.03+0.03) relationship (iii) sigma=0.25 - this could be a build factor,
where it has to be LOWER than the upper value for a hypothetical builder/builder
(sigma: 0.35=(0.3+0.4)/2) relationship; (iv) n=5 - a HIGHER n INCREASES the
"perceived" relationship, hence we use year as period because it does NOT
produce a HIGHER relationship value; (v) T=5 - as it is "inherently" risky to
co-invest with family, a HIGHER T creates a HIGHER relationship value that has
to be offset by rf.

```{.python .input}
%%R
str(filial.bscd)
BinomialTreePlot(filial.bscd$sRate)
BinomialTreePlot(filial.op$ce)
BinomialTreePlot(filial.op$pe)
```

```{.python .input}
%%R
BscdSaMtx <- function(model, S, X, type, row.seq=seq(0.01, 0.05, by=0.01), col.seq=seq(1.0, 0.7, by=-0.05))
{
  sMtx <- matrix(0, nrow=length(row.seq), ncol=length(col.seq))
  for( vRow in seq_along(row.seq) )
  {
    sa.bscd <- BscdCreateModel(row.seq[vRow], model$b, model$n, model$Time, model$sigma)
    for( wCol in seq_along(col.seq) )
    {
      sa.op           <- BscdOptionPrice( sa.bscd, S*col.seq[wCol], X, TypeFlag=type )
      sMtx[vRow,wCol] <- eval(parse(text=paste0("sa.op$",type,"[1,1]")))
    }
  }
  sMtx
}
ceMtx <- BscdSaMtx(filial.bscd, S=100, X=100, "ce")
peMtx <- BscdSaMtx(filial.bscd, S=100, X=100, "pe")
round(ceMtx,1)
round(peMtx,1)
```

### 1.2.1) (size: 26) It is interesting to note that the (European) put value
(23.8) is HIGHER than the call value (19.16). This implies that this joint
venture will most likely result in a DECREASE in relationship value, due to the
LOWER offset factor (rf), which takes into account the "inherent" risk of co-
investing with family over a longer time (T).
### 1.2.2) The rf (0.01) is, to a
large extent, a subjective estimate and thus it warrants a sensitivity analysis
about this value. We show the result of relationship value sensitivity in
relation to offset factor (rf) and asset value (S). The analysis of BOTH put and
call options (European) showed that rf has to INCREASE by 0.01 in order to have
EQUAL relationship values for BOTH put and call options. 
### 1.2.3) We have to
proactively identify parameters that could potentially INCREASE the relationship
value: (i) n - changing the number of periods does NOT alter the outcome of the
relationship value; (ii) T - the time to maturity could be LOWERED - by
disposing the property BEFORE TOP - to ENHANCE the relationship value, but this
event is very unlikely; (iii) sigma - the build factor has to INCREASE in order
for the relationship value to IMPROVE, but this parameter depends on EACH
person's LP, which is a fixed value; (iv) cc - the maintenance cost could be
LOWERED - by INCREASING the quality, NOT the quantity, of interaction - to
ENHANCE the relationship value.

```{.python .input}
%%R
rf          <- 0.03
cc          <- 0.05
sigma       <- 0.4
top.step    <- 5
top.year    <- 5
wp.bscd <- BscdCreateModel(rf, cc, top.step, top.year, sigma)
wp.op   <- BscdOptionPrice(wp.bscd, S=100, X=100)
wp.ce   <- BscdSaMtx(wp.bscd, S=100, X=100, "ce")
wp.pe   <- BscdSaMtx(wp.bscd, S=100, X=100, "pe")
round(wp.ce,1)
round(wp.pe,1)
```

### 1.3.1) (size: ) The SECOND model is for a relationship between Worker's
Party (WP) elected MP, Sylvia Lim, and I. With the future of my kids at stake, I
have considered performing voluteer work for an organisation that will bring
benefits to Singapore and to its people. The positions that I can contribute
"best" are technical positions, such as IT data analyst, or Financial / economic
data analyst: (i) rf=0.03 - this could be a "perceived" probability of success
OR an  offset factor, where it has to offset ALL other parameters (default:
0.03); (ii) c=0.05 - this could be a maintenance factor, where it has to be
LOWER than the upper value for a hypothetical insincere/insincere (c:
0.06=0.03+0.03) relationship (iii) sigma=0.4 - this could be a build factor,
where it has to be LOWER than OR EQUAL to the upper value for a hypothetical
builder/builder (sigma: 0.4=(0.4+0.4)/2) relationship; (iv) n=5 - a HIGHER n
INCREASES the "perceived" relationship, hence we use year as period because it
does NOT produce a HIGHER relationship value; (v) T=5 - this could be a period
between general elections for Singapore, OR it could be closely linked to
"inherent" risk, a HIGHER T creates a HIGHER relationship value that has to be
offset by rf.

```{.python .input}
%%R
rf          <- 0.03
cc          <- 0.02
sigma       <- 0.35
top.step    <- 5
top.year    <- 3
mod.bscd  <- BscdCreateModel(rf, cc, top.step, top.year, sigma)
mod.op    <- BscdOptionPrice(mod.bscd, S=100, X=100)
mod.ce    <- BscdSaMtx(mod.bscd, S=100, X=100, "ce")
mod.pe    <- BscdSaMtx(mod.bscd, S=100, X=100, "pe")
round(mod.ce,1)
round(mod.pe,1)
agg.bscd  <- BscdCreateModel(rf, cc+0.02, top.step, top.year, sigma-0.1)
agg.op    <- BscdOptionPrice(agg.bscd, S=100, X=100)
agg.ce    <- BscdSaMtx(agg.bscd, S=100, X=100, "ce")
agg.pe    <- BscdSaMtx(agg.bscd, S=100, X=100, "pe")
round(agg.ce,1)
round(agg.pe,1)
```

### 1.4.1) (size: ) The THIRD model is for a relationship between a hypothetical
client with MODERATE (AGGRESSIVE) risk tolerance and I, Dennis Lee: (i) rf=0.03
- this could be a "perceived" probability of success OR an  offset factor, where
it has to offset ALL other parameters (default: 0.03); (ii) c=0.02(0.04) - this
could be a maintenance factor, where it has to be LOWER than the upper value for
a hypothetical insincere/insincere (c: 0.06=0.03+0.03) relationship (iii)
sigma=0.35(0.25) - this could be a build factor, where it has to be LOWER than
OR EQUAL to the upper value for a hypothetical builder/builder (sigma:
0.4=(0.4+0.4)/2) relationship; (iv) n=5 - a HIGHER n INCREASES the "perceived"
relationship, hence we use year as period because it does NOT produce a HIGHER
relationship value; (v) T=3(3) - this could be a period between general
elections for Singapore, OR it could be closely linked to "inherent" risk, a
HIGHER T creates a HIGHER relationship value that has to be offset by rf.

```{.python .input}
%%R
rf          <- 0.03
cc          <- 0.05
sigma       <- 0.4
top.step    <- 12
top.year    <- 1
mod.bscd  <- BscdCreateModel(rf, cc, top.step, top.year, sigma)
mod.op    <- BscdOptionPrice(mod.bscd, S=100, X=100)
mod.ce    <- BscdSaMtx(mod.bscd, S=100, X=100, "ce")
mod.pe    <- BscdSaMtx(mod.bscd, S=100, X=100, "pe")
round(mod.ce,1)
round(mod.pe,1)
agg.bscd  <- BscdCreateModel(rf, cc+0.02, top.step, top.year, sigma-0.1)
agg.op    <- BscdOptionPrice(agg.bscd, S=100, X=100)
agg.ce    <- BscdSaMtx(agg.bscd, S=100, X=100, "ce")
agg.pe    <- BscdSaMtx(agg.bscd, S=100, X=100, "pe")
round(agg.ce,1)
round(agg.pe,1)
```

### 1.5.1) (size: ) The FOURTH model is for a relationship between Lim & Tan's
Business Manager, Lionel, and I. The job position that I applied for is mobile
broker.
### 1.5.2) c=0.05 - this could be a maintenance factor, where it has to
be LOWER than the upper value for a hypothetical insincere/insincere, (c:
0.05=0.02+0.03). Lionel's sincerity is based on "perceived" knowledge of
position, but is insincere about older workers, and expected success rate.
###
1.5.3) sigma=0.4 - this could be a build factor, where it has to be LOWER than
OR EQUAL to the upper value for a hypothetical builder/builder (sigma:
0.4=(0.4+0.4)/2) relationship; 
### 1.5.4) n=12 - a HIGHER n INCREASES the
"perceived" relationship, hence we use year as period because it does NOT
produce a HIGHER relationship value; 
### 1.5.5) T=1 - this could be a period of
"honeymoon" it could be closely linked to "inherent" risk, a HIGHER T creates a
HIGHER relationship value that has to be offset by rf.
### 1.5.6) rf=0.03 - this
could be a "perceived" probability of success OR an  offset factor, where it has
to offset ALL other parameters (default: 0.03).

```{.python .input}
%%R
rf          <- 0.04
cc          <- 0.04
sigma       <- 0.2
top.step    <- 12
top.year    <- 4
mod.bscd  <- BscdCreateModel(rf, cc, top.step, top.year, sigma)
mod.op    <- BscdOptionPrice(mod.bscd, S=100, X=100)
mod.ce    <- BscdSaMtx(mod.bscd, S=100, X=100, "ce", row.seq=seq(0.01, 0.09, by=0.01))
mod.pe    <- BscdSaMtx(mod.bscd, S=100, X=100, "pe", row.seq=seq(0.01, 0.09, by=0.01))
round(mod.ce,1)
round(mod.pe,1)
agg.bscd  <- BscdCreateModel(rf, cc+0.02, top.step, top.year-2, sigma+0.2)
agg.op    <- BscdOptionPrice(agg.bscd, S=100, X=100)
agg.ce    <- BscdSaMtx(agg.bscd, S=100, X=100, "ce", row.seq=seq(0.01, 0.09, by=0.01))
agg.pe    <- BscdSaMtx(agg.bscd, S=100, X=100, "pe", row.seq=seq(0.01, 0.09, by=0.01))
round(agg.ce,1)
round(agg.pe,1)
```

### 1.6.1) (size: ) The FIFTH model is for a relationship between a hypothetical
MT4 client with MODERATE (AGGRESSIVE) coder experience and I, Dennis Lee. 
###
1.6.2) c=0.04(0.06) - this could be a maintenance factor, where it has to be
LOWER than the upper value for a hypothetical insincere/insincere, (c:
0.06=0.03+0.03). E.g. for moderate clients: (a) incorrectly "perceived" to pay
MORE for technical support; (b) incorrectly "perceived" to NOT pirate code;
###
1.6.3) sigma=0.2(0.4) - this could be a build factor, where it has to be LOWER
than OR EQUAL to the upper value for a hypothetical builder/builder (sigma:
0.4=(0.4+0.4)/2) relationship; 
### 1.6.4) n=12 - a HIGHER n INCREASES the
"perceived" relationship, hence we use months as period because it does NOT
produce a HIGHER relationship value; 
### 1.6.5) T=4(2) - this could be a period
that creates "long-term" value with clients, OR it could be closely linked to
"inherent" risk, a HIGHER T creates a HIGHER relationship value that has to be
offset by rf. E.g. for moderate clients: (a) there is NO inherent risk of
family/friends; (b) willing to pay MORE for BETTER technical support.
### 1.6.6)
rf=0.05 - this could be a "perceived" probability of success OR an offset
factor, where it has to offset ALL other parameters (default: 0.03).
### 1.6.7)
It is interesting to note that the (European) put value is HIGHER than the call
value for BOTH moderate AND aggressive clients at rf=0.03 (default). This
implies that this sales venture will most likely result in a DECREASE in
relationship value, due to the HIGHER maintenance factor (cc), which takes into
account the "insincere" risk of investing with a complete stranger, however this
should get better over time.
### 1.6.8) The rf is, to a large extent, a
subjective estimate and thus it warrants a sensitivity analysis about this
value. We show the result of relationship value sensitivity in relation to
offset factor (rf) and asset value (S). The analysis of BOTH put and call
options (European) showed that rf has to INCREASE (moderate: +0.01; aggressive:
+0.03) from rf=0.03 (default) in order to have EQUAL relationship values for
BOTH put and call options.
### 1.6.9) The pricing plan is as follows: (a) MQL
code with NO plan: US$ 325; (b) MQL code with TWO(2)-year support plan: US$ 325
+ 75, where support plan includes email support, minor version upgrades (and
discounts on major upgrades). E.g. for aggressive client over 5 years is $1,925
( = 325 + 350 + 375 + 400 + 475 ), while for moderate client (with $150 discount
for major upgrades) over 5 years is $1,650 ( = 400 + 275 + 300 + 325 + 350 ).
Note: Due to the rf offset, there will be a "honeymoon" discount of US$ 100,
hence the MQL code will be "downsized" appropriately.
### 1.6.10) The evaluation
plan is as follows: (i) customized compiled restricted expert advisor; (ii) mql
code documentation; (iii) downloadable demo installer; (iv) create links for
"honeymoon" payments via Paypal that expires in SIXTY (60) days.
### 1.6.11) The
delivery plan is as follows: (i) "downsized" MQL code; (ii) mql code
documentation; (iii) agreement contract; (iv) tax invoice; and (v) downloadable
installer;
