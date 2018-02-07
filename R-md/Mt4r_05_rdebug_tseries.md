Mt4r_05_rdebug_tseries
==================================================================================================================================
## Motivational Buckets
### 1.1) (size: ) The motivation comes from Coursera's
Stanford Online Machine Learning (STML) course by Professor Andrew Ng, in
particular the Gradient Descent (GD) method to minimize a cost function. For
least square regression model (lm), the cost function is represented by
J(b0,b1), where b1 is the intercept (coefficient 0) and b2 is the slope
(coefficient 1), that is a convex-shaped curve.
### 1.2) (size: )
Mathematically, the formula to obtain the minimum J(b0,b1) is given by TWO (2)
"simultaneous" equations: (a) b0 = b0 - alpha * dJ()/dx; (b) b1 = b1 - alpha *
dJ()/dx, where alpha is the rate of descent and dJ()/dx is the first
differential equation (FDE) of the cost function J.
### 1.3) (size: ) When alpha
is too small, the GD method may converge too slowly, but when alpha is too
large, the GD method may NOT converge. In the case of least square regression,
the FDE of J is the slope of tangent-line to the convex-shaped curve. Hence,
when the slope is positive, the equations above result in smaller values, e.g.
b0 = (b0 - positive number), but when the slope is negative, the results are
larger values, e.g. b1 = (b1 - negative number). Eventually, when the slope is
ZERO (0), the results do NOT change.
### 1.4) (size: ) The idea is to extend the
GD method to create a volatility indicator based on close price. For a given
range of prices, we build a lm and return the slope, which is analogous to the
FDE above. Therefore, when the slope is positive, the formula to obtain the
minimum volatility is b0 = (b0 - positive number), but when the slope is
negative, the formula to obtain the minimum volatility is b0 = (b0 - negative
number). The minimum volatility is when the slope is ZERO (0), the price is a
support/resistance price.
### 1.5) (size: ) When parameter alpha is too small,
the indicator converges slowly to the support/resistance price, but when alpha
is too large, the indicator may overshoot the support/resistance price. The
other parameter is the width (or window size) of the price range. When the width
is too small, the slope may be volatile, but when the width is too large, the
slope may be "smoothed" too much.

```{.python .input}
%%R
if( Sys.info()["sysname"] == "Linux" )
  suppressPackageStartupMessages(source("~/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE))
if( Sys.info()["sysname"] == "Windows" )
  suppressPackageStartupMessages(source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE))
suppressPackageStartupMessages(source(paste0(RegRSourceDir(),"PlusFile.R"), echo=FALSE))
suppressPackageStartupMessages(source(paste0(RegRSourceDir(),"PlusMtrDevice.R"), echo=FALSE))
suppressPackageStartupMessages(library(R.utils))
suppressPackageStartupMessages(library(tseries))
name.str    <- "GradientVolatility"
ver.str     <- "0.1.0"
mt4.str     <- "MT4 Go 001/"
exe.dir     <- paste0(RegProgramDir(), mt4.str)
exe.str     <- "metalang.exe"
ind.dir     <- paste0(RegLocalProgramDir(), mt4.str, "experts/indicators/")
mqh.dir     <- paste0(RegLocalProgramDir(), mt4.str, "experts/include/")
MtrDeviceWriterStr(mqh.dir)
```

```{.python .input}
%%R
bufNum        <- 2
styleChr      <- c('DRAW_LINE','DRAW_NONE')
lType         <- c(rep('property', 6))
lName         <- c('indicator_buffers','indicator_color1','indicator_color2',
                   'indicator_minimum','indicator_maximum',
                   'indicator_separate_window')
lVal          <- c(bufNum,'Red','Black','-1','1','')
eType         <- c('int','int','double')
eName         <- c('GdvPeriod','GdvLookBack','GdvAlpha')
eVal          <- c('20','500','1.0')
mt.list       <- MtrAddRdeviceTop(name.str,ver.str,lType,lName,lVal,eType,eName,eVal)
mt.list       <- append( mt.list, MtrAddRdeviceInit(bufNum,styleChr,rep(eName[1],bufNum),
                                                    c('zoo'), c('PlusReg.R')) )
mt.list       <- append( mt.list, MtrAddRStart() )
mt.list       <- append( mt.list, MtrAddRdeviceDeinit() )
```

```{.python .input}
%%R
mt.chr        <- MtrWriterStr(mt.list)
names(mt.chr) <- NULL
mt.chr
ea.dir.str  <- MtrEaWriterStr(name.str, mt.list, save.dir=ind.dir)
cmd.str     <- paste0('"', exe.dir, exe.str, '" "', ea.dir.str, '"')
cmd.str
errChr      <- suppressWarnings(system(cmd.str, intern=TRUE, 
                                       wait=TRUE, show.output.on.console=FALSE))
out.list    <- strsplit(errChr, ";", fixed=TRUE)
err.list    <- out.list[which(lapply(out.list, "[", c(1))=="2")]
warn.list   <- out.list[which(lapply(out.list, "[", c(1))=="1")]
length(err.list)
head(err.list)
length(warn.list)
head(warn.list)
```
