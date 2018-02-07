Mt4r_04_graddesc_volatility_tseries
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
suppressPackageStartupMessages(source(paste0(RegRSourceDir(),"PlusMtr.R"), echo=FALSE))
suppressPackageStartupMessages(library(R.utils))
suppressPackageStartupMessages(library(tseries))
name.str      <- "GradientVolatility0"
ver.str       <- "0.1.0"
exe.dir     <- paste0(RegProgramDir(), "Go MT4 001/")
exe.str     <- "metalang.exe"
ind.dir     <- paste0(RegLocalProgramDir(), "Go MT4 001/experts/indicators/")
```

```{.python .input}
%%R
MtrAddTop <- function(linkType, linkName, linkVal, extType, extName, extVal)
{
  top <- list(c('//:::::::::::::::::::::::::::::::::::::::::::::'))
  for( i in seq_along(linkType) )
    top <- append(top, list(c(paste0('#',linkType[i]),linkName[i],linkVal[i])))
  for( i in seq_along(extType) )
    top <- append(top, list(c('extern',extType[i],extName[i],'=',paste0(extVal[i],';'))))
  
  end <- list(c('//:::::::::::::::::::::::::::::::::::::::::::::'))
  
  return( append(top,end) )
}
MtrAddInit <- function(nameStr, verStr, bufNum, styleChr=NULL, drawBegin=NULL)
{
  if( is.null(styleChr) )
    styleChr  <- rep( 'DRAW_LINE', bufNum )
  if( is.null(drawBegin) )
    drawBegin <- rep( 0, bufNum )
  
  top <- list(c('//:::::::::::::::::::::::::::::::::::::::::::::'),
              c('string','IndName=',paste0('\"',nameStr,'\";')),
              c('string','IndVer=',paste0('\"',verStr,'\";')))
  
  for( i in 1:bufNum )
    top <- append(top, list(c('double',paste0('ExtMapBuffer',i,'[];'))))

  mid <- list(c('int','init()'),
              c('{'),
              cs(2,paste0('IndicatorBuffers(',bufNum,');')),
              cs(2,'IndicatorDigits(Digits+10);'),
              cs(2,'IndicatorShortName(StringConcatenate(IndName," ",IndVer));'))
  names(mid)[1] <- "init"
  
  for( i in 1:bufNum )
    mid <- append(mid, list(cs(2,'SetIndexStyle(',i-1,',',styleChr[i],');')))
  for( i in 1:bufNum )
    mid <- append(mid, list(cs(2,'SetIndexDrawBegin(',i-1,',',drawBegin[i],');')))
  for( i in 1:bufNum )
    mid <- append(mid, list(cs(2,'SetIndexBuffer(',i-1,',',paste0('ExtMapBuffer',i),');')))

  end <- list(cs(2,'return(0);'), 
              c('}'),
              c('//:::::::::::::::::::::::::::::::::::::::::::::'))
  names(end)[2] <- "init-end"
  
  ret <- append(top, mid)
  ret <- append(ret, end)
              
  return( ret )
}
MtrAddDeinit <- function()
{
  ret <- list(c('//:::::::::::::::::::::::::::::::::::::::::::::'),
              c('int','deinit()'),
              c('{'),
              cs(2,'return(0);'), 
              c('}'),
              c('//:::::::::::::::::::::::::::::::::::::::::::::'))
  names(ret)[2] <- "deinit"
  names(ret)[5] <- "deinit-end"

  return( ret )
}
MtrAddStart <- function()
{
  top <- list(c('int','start()'),
              c('{'),
              cs(2,'int','i;'),
              cs(2,'int','unused_bars;'),
              cs(2,'int','used_bars=IndicatorCounted();'),
              c(''),
              cs(2,'if','(used_bars<0)','return(-1);'),
              cs(2,'if','(used_bars>0)','used_bars--;'),
              cs(2,'unused_bars=Bars-used_bars;'),
              c(''))
  names(top)[1] <- "start"
  
  mid <- list(cs(2,'for(i=unused_bars-1;i>=0;i--)'),
              cs(2,'{'),
              cs(2,'}'))

  end <- list(cs(2,'return(0);'), 
              c('}'),
              c('//:::::::::::::::::::::::::::::::::::::::::::::'))
  names(end)[2] <- "start-end"

  #ret <- append(top, mid)
  ret <- append(top, end)
              
  return( ret )
}
MtrAddInVar <- function(varType, varName, indent=0)
{
  ret <- list()
  for( i in seq_along(varType) )
    ret <- append(ret, list(c(rep('',indent),varType[i],varName[i],';')))
  ret
}
MtrAddInModel <- function()
{
  ret <- list(cs(2,'ArrayResize(hist,unused_bars-1);'),
              cs(2,'for(i=unused_bars-2;i>=0;i--)'),
              cs(2,'{'),
              cs(4,'hist[i]','=','Close[i];'),
              cs(2,'}'),
              cs(2,MtrAssignVector0("hNum",',hist,ArraySize(hist)')),
              cs(2,MtrExecute0("hNum <- rev(hNum)")),
              cs(2,MtrExecute0("histNum <- c(histNum,hNum)")),
              cs(2,MtrExecuteAsync0("model <- rollapply(histNum,20,function(x) as.numeric(lm(x ~ seq_along(x))$coeff[2]))*1000")))
   
  return( ret )
}
MtrAddInResult <- function()
{
  cmd <- paste0("as.integer(exists(",pasteq0("model"),"))")
  ret <- list(cs(2,'ArrayResize(ret,GdvLookBack);'),
              cs(2,'int','len=',MtrGetInteger0("length(histNum)")),
              cs(2,'if(',MtrGetInteger(cmd),'==1)'),
              cs(2,'{'),
              cs(4,'RGetVector(R, StringConcatenate(',pasteq("rev(model)[1:"),',GdvLookBack,',pasteq("]"),'),ret,GdvLookBack);'),
              cs(4,'for(i=0;i<GdvLookBack;i++)'),
              cs(4,'{'),
              cs(6,'ExtMapBuffer1[i]','=','ret[i];'),
              cs(4,'}'),
              cs(2,'}'))
  
  return( ret) 
}
MtrEaWriterStr <- function(name.str, mt.list, save.dir=RegHomeDir())
{
  #---  Check that arguments are valid
  stopStr <- AddAvoidN(name.str)
  if( !is.null(stopStr) ) stop(stopStr)
  stopStr <- AddAvoidN(mt.list)
  if( !is.null(stopStr) ) stop(stopStr)
  stopStr <- AddExistN(substr(save.dir,1,nchar(save.dir)-1))
  if( !is.null(stopStr) ) stop(stopStr)
  
  ea.str  <- paste0(name.str, ".mq4")
    
  #---  Write data
  #       Write EACH node of the list as a line
  #       Separate the elements of EACH node with a space.
  fCon    <-file(paste0(save.dir,ea.str))
  writeLines(unlist(lapply(mt.list, paste, collapse=" ")), fCon)
  close(fCon)
  return( paste0(save.dir,ea.str) )
}
pasteq  <- function(x) paste0("\"",x,"\"")
pasteq0 <- function(x) paste0("\'",x,"\'")
cs      <- function(n, ...) c(rep('',n), ...)
MtrExecute0         <- function(x) c('RExecute(R,',pasteq(x),');')
MtrExecuteAsync0    <- function(x) c('RExecuteAsync(R,',pasteq(x),');')
MtrGetInteger       <- function(x) c('RGetInteger(R,',pasteq(x),')')
MtrGetInteger0      <- function(x) c('RGetInteger(R,',pasteq(x),');')
MtrGetVector0       <- function(x,...) c('RGetVector(R,',pasteq(x),...,');')
MtrAssignVector0    <- function(x,...) c('RAssignVector(R,',pasteq(x),...,');')
```

```{.python .input}
%%R
bufNum        <- 2
styleChr      <- c('DRAW_LINE','DRAW_NONE')
lType         <- c(rep('property', 6),'include')
lName         <- c('indicator_buffers','indicator_color1','indicator_color2',
                   'indicator_minimum','indicator_maximum',
                   'indicator_separate_window','<mt4R.mqh>')
lVal          <- c(bufNum,'Red','Black','-1','1','','')
eType         <- c('int','int','double','string')
eName         <- c('GdvPeriod','GdvLookBack','GdvAlpha','Rpath')
eVal          <- c('20','500','1.0',pasteq('C:/Program Files/R/R-2.15.3/bin/i386/Rterm.exe'))
mt.list       <- MtrAddTop(lType,lName,lVal,eType,eName,eVal)
mt.list       <- append( mt.list, MtrAddInit(name.str,ver.str,bufNum,
                                             styleChr,rep(eName[1],bufNum)) )
mt.list       <- append( mt.list, MtrAddStart() )
mt.list       <- append( mt.list, MtrAddDeinit() )
```

```{.python .input}
%%R
ins.list  <- MtrAddInVar('int','R')
mt.list   <- append( mt.list, ins.list, 
                     after=which(names(mt.list)=="init")-1 )
ins.list  <- list(cs(2,'string','Rterm','=','StringConcatenate(Rpath,',pasteq(" --no-save"),');'),
                  cs(2,'R=Rinit(Rterm,2);'),
                  cs(2,MtrExecute0("histNum <- numeric(0)")),
                  cs(2,MtrExecute0("library(zoo)")))
                  
mt.list   <- append( mt.list, ins.list,
                     after=which(names(mt.list)=="init-end")-2 )
ins.list  <- list(cs(2,'RDeinit(R);'))
mt.list   <- append( mt.list, ins.list,
                     after=which(names(mt.list)=="deinit-end")-2 )
ins.list  <- MtrAddInVar(rep('double',2),
                         c('hist[]','ret[]'), indent=2)
mt.list   <- append( mt.list, ins.list, 
                     after=which(names(mt.list)=="start")+2 )
ins.list  <- list(cs(2,'if(RIsBusy(R))','return(0);'))
ins.list  <- append( ins.list, MtrAddInResult() )
ins.list  <- append( ins.list, MtrAddInModel() )
mt.list   <- append( mt.list, ins.list,
                     after=which(names(mt.list)=="start-end")-2 )

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
