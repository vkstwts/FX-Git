Mt4r_03_Keiji_probot_tseries
==================================================================================================================================
## Motivational Buckets
### 1.1) (size: ) Keiji is the author of a script,
namely "mt4to5rewrite_sample_v4_2.mq4", that converts MT4 code into MT5 code.
This R Markdown file bases the main function on this script, in particular the
start() function, but rather to convert MT4 to SqLite. 
### 1.2) (size: ) There
are TWO (2) approaches to convert MT4 to SqLite: (a) replace ALL order tokens
with the related SqLite tokens: (b) replace ANY command tokens - OrderSend,
OrderModify, OrderClose, OrderDelete - with SqLite command arrays, if the
command tokens are embedded within a query token - OrderSelect. In order to
perform the latter approach, we MUST first build a knowledge data set of the
existing code.
### 1.3) (size: ) There are SIX (6) internal functions: (i)
FuncAddTop(); (ii) FuncAddInInit(); (iii) FuncAddInStart(); (iv)
FuncAddInDeinit(); (v) FuncFind(); (vi) FuncRewrite().
### 1.4) (size: ) There
are FIVE (5) include files: (i) "mt4accountinfo.mqh"; (ii) "mt4string.mqh";
(iii) "mt4datetime.mqh"; (iv) "mt4objects_1.mqh"; and (v) "mt4timeseries_2.mqh".
### 1.5) (size: ) We start with the end goal: (i) convert the mt.list (java)
into mq4; (ii) add codes for include, init, deinit, and refresh (optionally
comment); (iii) replace ALL order tokens with the related SqLite tokens; (iv)
replace ANY command tokens with SqLite command arrays;

```{.python .input}
%%R
if( Sys.info()["sysname"] == "Linux" )
  suppressPackageStartupMessages(source("~/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE))
if( Sys.info()["sysname"] == "Windows" )
  suppressPackageStartupMessages(source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE))
suppressPackageStartupMessages(source(paste0(RegRSourceDir(),"PlusFile.R"), echo=FALSE))
suppressPackageStartupMessages(source(paste0(RegRSourceDir(),"PlusMtrGhost.R"), echo=FALSE))
suppressPackageStartupMessages(library(R.utils))
suppressPackageStartupMessages(library(tseries))
name.str      <- "Pro_Bot_EURUSD_m15_Risk"
java.dir.str  <- MtrConvertStr(name.str)
exe.dir       <- paste0(RegProgramDir(), "Go MT4 001/")
exe.str       <- "metalang.exe"
if( file.exists(java.dir.str) )
  mt.list <- lapply(readLines(java.dir.str), function(x) unlist(strsplit(x," ",fixed=TRUE)))
```

### 1.1.1) (size: ) In this instance, we "normalize" the Expert Advisor (EA)
named "Pro_Bot_EURUSD_m15_Risk.mq4" into a java file
"Pro_Bot_EURUSD_m15_Risk.java" by using the executable file "mq4_writer.exe".
"Normalize" means to clean up and standardize the format of the code without
changing its implementation. Ideally, we would have preferred to "normalize" the
EA into a MQ4 file but there is no utility to perform this.
### 1.1.2) After the
normalization process, we then read the java file into a list, where an element
of the list corresponds to a line in the text file, and EACH element contains a
vector of characters (including indentation).

```{.python .input}
%%R
mt.list <- MtrTagInJClass(mt.list, name.str)
mt.list <- MtrTagInJLink(mt.list)
mt.list <- MtrTagInJExtern(mt.list)
cmtDfr  <- MtrFindCmtDfr(mt.list)
cmtDfr
forDfr  <- MtrFindLoopDfr(mt.list, cmtDfr, tokenChr=c("for", "while", "do"))
forDfr
funDfr  <- MtrFindFunDfr(mt.list, cmtDfr)
fuaDfr  <- funDfr[funDfr$Name!="init", ]
fuaDfr  <- fuaDfr[fuaDfr$Name!="start", ]
fuaDfr  <- fuaDfr[fuaDfr$Name!="deinit", ]
fuaDfr
fubDfr  <- MtrBetweenFunDfr(mt.list, fuaDfr, cmtDfr)
fubDfr
fobDfr  <- MtrBetweenLoopDfr(mt.list, forDfr, cmtDfr, as.character(fubDfr$Name))
fobDfr
mt.list <- MtrTagInJFun(mt.list, funDfr)
mt.list <- MtrTagInJGvar(mt.list, funDfr)
mt.list <- MtrJConvert(mt.list, funDfr)
mt.list <- MtrJType(mt.list)
MtrWriterStr(mt.list)
MtrAddTop <- function()
{
  return(list(c('//:::::::::::::::::::::::::::::::::::::::::::::'),
              c('#include','<PlusTurtle.mqh>'),
              c('#include','<PlusGhost.mqh>'),
              c('int','MaxAccountTrades=4'),
              c('//:::::::::::::::::::::::::::::::::::::::::::::')));
}
MtrAddInInit <- function()
{
  return(list(c('//:::::::::::::::::::::::::::::::::::::::::::::'),
              c('TurtleInit();'),
              c('GhostInit();'),
              c('//:::::::::::::::::::::::::::::::::::::::::::::')));
}
MtrAddInDeinit <- function()
{
  return(list(c('//:::::::::::::::::::::::::::::::::::::::::::::'),
              c('GhostDeInit();'),
              c('//:::::::::::::::::::::::::::::::::::::::::::::')));
}
MtrAddInStart <- function()
{
  return(list(c('//:::::::::::::::::::::::::::::::::::::::::::::'),
              c('GhostRefresh();'),
              c('//:::::::::::::::::::::::::::::::::::::::::::::')));
}
MtrArrayTop <- function()
{
  return(list(c('int','aCommand[];'),    
              c('int','aTicket[];'),
              c('double','aLots[];'),
              c('double','aClosePrice[];'),
              c('bool','aOk;'),
              c('int','aCount;'),
              c('int','total;'),
              c('ArrayResize(aCommand,maxAccountTrades);'),
              c('ArrayResize(aTicket,maxAccountTrades);'),
              c('ArrayResize(aLots,maxAccountTrades);'),
              c('ArrayResize(aClosePrice,maxAccountTrades);')))
}
MtrArrayBottom <- function()
{
  return(list(c('for(int','i=0;','i<aCount;','i++)'),
              c('{'),
              c('','','switch(aCommand[i])'),
              c('','','{'),
              c('','','','','case','2:'),
              c('','','','','','','OrderClose(aTicket[i],0,aLots[i]);'),
              c('','','','','','','break;'),
              c('','','','','case','4:'),
              c('','','','','','','OrderClose(aTicket[i],1,aLots[i]);'),
              c('','','','','','','break;'),
              c('','','}'),
              c('}')))
}
MtrArrayInit <- function()
{
  return(list(c('aCommand[aCount]=0;'),
              c('aTicket[aCount]=GhostOrderTicket();'),
              c('aLots[aCount]=GhostOrderLots();'),
              c('aClosePrice[aCount]=GhostOrderClosePrice();')))
}
MtrArrayOrder <- function()
{
  return(list(c('aCommand[aCount]=2;'),
              c('aCount++;'),
              c('if(aCount>=maxAccountTrades)','break;')))

}
MtrSelectInit <- function()
{
  return(list(c('GhostInitSelect(true,0,SELECT_BY_POS,MODE_TRADES);')))
}
MtrSelectFree <- function()
{
  return(list(c('GhostFreeSelect(false);')))
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
  
  ea.str  <- paste0(name.str, "_SqLite.mq4")
    
  #---  Write data
  #       Write EACH node of the list as a line
  #       Separate the elements of EACH node with a space.
  fCon    <-file(paste0(save.dir,ea.str))
  writeLines(unlist(lapply(mt.list, paste, collapse=" ")), fCon)
  close(fCon)
  return( paste0(save.dir,ea.str) )
}
```

```{.python .input}
%%R
sql.list <- list()
j <- 1
for( i in seq_along(mt.list) )
{
  #--- Read Line
  lineChr <- mt.list[[i]]
  
  #--- Account Query
  lineChr <- gsub("AccountFreeMargin", "GhostAccountFreeMargin", lineChr)
  
  #--- Data types
  lineChr <- gsub("Date",   "datetime", lineChr)
  lineChr <- gsub("String", "string", lineChr)
  
  #--- Sql Query  
  lineChr <- gsub("OrderTicket",      "GhostOrderTicket", lineChr)
  lineChr <- gsub("OrderSymbol",      "GhostOrderSymbol", lineChr)
  lineChr <- gsub("OrderOpenPrice",   "GhostOrderOpenPrice", lineChr)
  lineChr <- gsub("OrderMagicNumber", "GhostOrderMagicNumber", lineChr)
  lineChr <- gsub("OrderType",        "GhostOrderType", lineChr)
  lineChr <- gsub("OrderStopLoss",    "GhostOrderStopLoss", lineChr)
  lineChr <- gsub("OrderTakeProfit",  "GhostOrderTakeProfit", lineChr)
  
  #--- Sql Query with special cases
  lineChr <- gsub("OrderSelect", "GhostOrderSelect", lineChr)
  subBln  <- length(grep("GhostOrdersSelect", lineChr)) > 0
  posBln  <- length(grep("SELECT_BY_POS", lineChr)) > 0
  tktBln  <- length(grep("SELECT_BY_TICKET", lineChr)) > 0
  trdBln  <- length(grep("MODE_TRADES", lineChr)) > 0
  hsyBln  <- length(grep("MODE_HISTORY", lineChr)) > 0
  if( posBln & subBln )
  {
    
  }
  if( tktBln & subBln )
  {
    
  }  
  
  lineChr <- gsub("OrdersTotal", "GhostOrdersTotal", lineChr)
  subBln  <- length(grep("GhostOrdersTotal", lineChr)) > 0
  forBln  <- length(grep("for", lineChr)) > 0
  if( forBln & subBln )
  {
    #--- Prepend a line
    sql.list[[j]] <- c("int", "total", "=", "GhostOrdersTotal();")
    j <- j + 1
    lineChr <- gsub("GhostOrdersTotal\\(\\)", "total", lineChr)
  }
  
  #--- Sql Command
  lineChr <- gsub("OrderSend",   "GhostOrderSend", lineChr)
  lineChr <- gsub("OrderModify", "GhostOrderModify", lineChr)
  
  #--- Write Line
  sql.list[[j]] <- lineChr
  j <- j + 1
}
```

### 1.3.1) (size: ) The internal function FuncRewrite() has an equivalent
function gsub() in R.

```{.python .input}
%%R
ea.dir.str    <- MtrEaWriterStr(name.str, mt.list)
cmd.str       <- paste0('"', exe.dir, exe.str, '" "', ea.dir.str, '"')
errChr        <- suppressWarnings(system(cmd.str, intern=TRUE,
                                         wait=TRUE, show.output.on.console=FALSE))
out.list      <- strsplit(errChr, ";", fixed=TRUE)
err.list      <- out.list[which(lapply(out.list, "[", c(1))=="2")]
warn.list     <- out.list[which(lapply(out.list, "[", c(1))=="1")]
length(err.list)
head(err.list)
length(warn.list)
head(warn.list)
```
