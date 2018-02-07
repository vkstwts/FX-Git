Mt4r_01_Keiji_mt4to5_tseries
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

```{.python .input}
%%R
if( Sys.info()["sysname"] == "Linux" )
{
  suppressPackageStartupMessages(source("~/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE))
}
if( Sys.info()["sysname"] == "Windows" )
{
  suppressPackageStartupMessages(source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE))
}
suppressPackageStartupMessages(source(paste0(RegRSourceDir(),"PlusFile.R"), echo=FALSE))
suppressPackageStartupMessages(library(R.utils))
suppressPackageStartupMessages(library(tseries))
MtrConvertStr <- function(name.str, exe.dir=paste0(RegProgramDir(),"mq4_converter/"),
                       ea.dir=RegEaDir(), java.dir=RegJavaDir())
{
  ea.str    <- paste0(name.str, ".mq4")
  java.str  <- paste0(name.str, ".java")
  exe.str   <- "mq4_writer.exe"
  cmd.str   <- paste0('"', exe.dir, exe.str, '" "', ea.dir, ea.str, 
                    '" java "', java.dir, java.str, '"')
  if( !file.exists(paste0(exe.dir, exe.str)) )
    stop(paste0(exe.str, ' MUST be installed first'))
  if( !file.exists(paste0(ea.dir, ea.str)) )
    stop(paste0(ea.str, ' file MUST exists'))
  errNum <- RegSystemNum(cmd.str)
  if( errNum!=0 | !file.exists(paste0(java.dir, java.str)) )
    return( paste0(errNum, ': ', java.str, ' is missing (OR NOT converted correctly)') )
  else
    return( paste0(java.dir, java.str) )
}
name.str  <- "Pro_Bot_EURUSD_m15_Risk"
#name.str  <- "Gday mark 2"
java.str  <- MtrConvertStr(name.str)
if( file.exists(java.str) )
  mt.list <- lapply(readLines(paste0(java.str)), function(x) scan(text=x, sep=" ", 
                                                 what=c("char"), strip.white=c(FALSE)))
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
head(mt.list)
MtrFindCmtDfr <- function(mt.list)
{
  endNum <- length(mt.list)
  retDfr <- dataFrame( colClasses=c(Token="character", Open="numeric", Close="numeric",
                                    First="numeric"), nrow=0 )
  rowNum <- which(lapply(mt.list, function(x) { sum(grep("//", x)) })>0)
  for( n in seq_along(rowNum) )
  {
    openNum     <- rowNum[n]
    indentNum   <- which(nchar(mt.list[[openNum]])>0)
    rDfr        <- data.frame("cmt", openNum, openNum, indentNum[1] )
    names(rDfr) <- names(retDfr)
    retDfr      <- rbind(retDfr, rDfr)
  }
  rowNum    <- which(lapply(mt.list, function(x) { sum(grep("\\/\\*", x)) })>0)
  rowcNum   <- which(lapply(mt.list, function(x) { sum(grep("\\*\\/", x)) })>0)
  for( n in seq_along(rowNum) )
  {
    openNum     <- rowNum[n]
    indentNum   <- which(nchar(mt.list[[openNum]])>0)
    closeNum    <- rowcNum[n]
    rDfr        <- data.frame("cmt", openNum, closeNum, indentNum[1] )
    names(rDfr) <- names(retDfr)
    retDfr      <- rbind(retDfr, rDfr)
  }
  retDfr
}
MtrIsComment <- function(tokenStr, rowNum, cmtDfr)
{
  retBln  <- NULL
  for( n in seq_along(rowNum) )
  {
    openNum     <- rowNum[n]
    #---  Identify non-valid tokens
    #     (1) tokens may be within a comment: (a) // ; (b) /*  */
    cDfr        <- cmtDfr[cmtDfr$Open<=openNum & openNum<=cmtDfr$Close,]
    isOpenCmt   <- nrow(cDfr)>0
    retBln <- c(retBln, isOpenCmt)
  }
  retBln
}
MtrFindLoopDfr <- function(mt.list, cmtDfr, tokenChr=c("for"))
{
  endNum <- length(mt.list)
  retDfr <- dataFrame( colClasses=c(Token="character", Open="numeric", 
                                    Close="numeric", First="numeric"), nrow=0 )
  for( m in seq_along(tokenChr) )
  {
    tokenStr  <- tokenChr[m]
    rowNum    <- which(lapply(mt.list, function(x) { sum(grep(tokenStr, x)) })>0)
    if(length(rowNum)>0)
      cmtBln    <- MtrIsComment(tokenStr, rowNum, cmtDfr)
    for( n in seq_along(rowNum) )
    {
      openNum     <- rowNum[n]
      indentNum   <- which(nchar(mt.list[[openNum]])>0)
      isOpenCmt   <- cmtBln[n]
      #---  Identify non-valid tokens
      #     (1) tokens may be within a string, i.e. " this is a string "
      if( length(indentNum)>1 )
        isOpenStr   <- substr(mt.list[[openNum]][indentNum[2]],1,1)!="("
      else
        isOpenStr   <- TRUE
      if( !isOpenCmt &  !isOpenStr )
      {
        #---  Knowledge when "for" has braces OR NOT
        #     (1) check if next token is "{"
        nextNum     <- openNum+1
        indnxtNum   <- which(nchar(mt.list[[nextNum]])>0)
        if( indentNum[1]==indnxtNum[1] )
          isOpenBrs   <- substr(mt.list[[nextNum]][indnxtNum[1]],1,1)=="{"
        else
          isOpenBrs   <- FALSE
        if( !isOpenBrs )
          startNum <- openNum + 1
        else
          startNum <- openNum + 2
        for( mRow in startNum:endNum )
        {
          iNum <- which(nchar(mt.list[[mRow]])>0)
          if( length(iNum)>0 )
            if( iNum[1]==indentNum[1] )
            {
              isCloseCmt  <- nrow(cmtDfr[cmtDfr$Open==mRow,])>0
              if( !isCloseCmt ) break
            }
        }
        if( !isOpenBrs ) 
          closeNum  <- mRow - 1
        else
          closeNum  <- mRow
        rDfr        <- data.frame(tokenStr, openNum, closeNum, indentNum[1] )
        names(rDfr) <- names(retDfr)
        retDfr      <- rbind(retDfr, rDfr)
      }
    }
  }
  retDfr
}
MtrFindFunDfr <- function(mt.list, cmtDfr, tokenChr=c("public"), offset=3)
{
  endNum <- length(mt.list)
  retDfr <- dataFrame( colClasses=c(Token="character", Open="numeric", 
                                    Close="numeric", First="numeric",
                                    Name="character"), nrow=0 )
  for( m in seq_along(tokenChr) )
  {
    tokenStr  <- tokenChr[m]
    rowNum    <- which(lapply(mt.list, function(x) { sum(grep(tokenStr, x)) })>0)
    cmtBln    <- MtrIsComment(tokenStr, rowNum, cmtDfr)
    for( n in seq_along(rowNum) )
    {
      openNum     <- rowNum[n]
      indentNum   <- which(nchar(mt.list[[openNum]])>0)
      isOpenCmt   <- cmtBln[n]
      #---  Identify non-valid tokens
      #     (1) tokens may be within a string, i.e. " this is a string "
      if( length(indentNum)>1 )
        isOpenStr   <- sum(grep("\\(", mt.list[[openNum]][indentNum[offset]]))==0
      else
        isOpenStr   <- TRUE
      if( !isOpenCmt &  !isOpenStr )
      {
        #---  Knowledge when "for" has braces OR NOT
        #     (1) check if next token is "{"
        nameStr     <- substr(mt.list[[openNum]][indentNum[offset]], 1,
                              as.numeric(gregexpr("\\(", mt.list[[openNum]][indentNum[offset]]))-1)
        nextNum     <- openNum+1
        indnxtNum   <- which(nchar(mt.list[[nextNum]])>0)
        if( indentNum[1]==indnxtNum[1] )
          isOpenBrs   <- substr(mt.list[[nextNum]][indnxtNum[1]],1,1)=="{"
        else
          isOpenBrs   <- FALSE
        if( !isOpenBrs )
          startNum <- openNum + 1
        else
          startNum <- openNum + 2
        for( mRow in startNum:endNum )
        {
          iNum <- which(nchar(mt.list[[mRow]])>0)
          if( length(iNum)>0 )
            if( iNum[1]==indentNum[1] )
            {
              isCloseCmt  <- nrow(cmtDfr[cmtDfr$Open==mRow,])>0
              if( !isCloseCmt ) break
            }
        }
        if( !isOpenBrs ) 
          closeNum  <- mRow - 1
        else
          closeNum  <- mRow
        rDfr        <- data.frame(tokenStr, openNum, closeNum, indentNum[1],
                                  nameStr)
        names(rDfr) <- names(retDfr)
        retDfr      <- rbind(retDfr, rDfr)
      }
    }
  }
  retDfr
}
MtrSubLoopDfr <- function(mt.list, loopDfr, cmtDfr, funThatChr=NULL)
{
  funChr <- c("OrderSend", "OrderModify", "OrderClose", funThatChr)
  endNum <- length(mt.list)
  retDfr <- dataFrame( colClasses=c(Token="character", Open="numeric", 
                                    Close="numeric", First="numeric"), nrow=0 )
  for( n in seq_along(loopDfr$Open) )
  {
    #---  Knowledge when "for" has OrderSelect and ONE (1) or more of OrderSend
    #     (1) check for OrderSelect
    #     (2) check for OrderSend
    #     (3) check for OrderModify
    #     (4) check for OrderClose
    #     (5) check for OrderDelete
    tokenStr  <- loopDfr$Token[n]
    openNum   <- loopDfr$Open[n]
    closeNum  <- loopDfr$Close[n]
    firstNum  <- loopDfr$First[n]
    rowNum    <- which(lapply(mt.list[openNum:closeNum],
                              function(x) { sum(grep("OrderSelect", x)) })>0)
    cmtBln    <- MtrIsComment("OrderSelect", rowNum+openNum-1, cmtDfr)
    isOpenSelect <- sum(cmtBln)<length(cmtBln)
    if( isOpenSelect )
    {
      isOpenFun   <- FALSE
      for( o in seq_along(funChr) )
      {
        funStr    <- funChr[o]
        fRowNum   <- which(lapply(mt.list[openNum:closeNum],
                                  function(x) { sum(grep(funStr, x)) })>0)
        fCmtBln   <- MtrIsComment(funStr, fRowNum+openNum-1, cmtDfr)
        isOpenFun <- isOpenFun | (sum(fCmtBln)<length(fCmtBln))
      }
      if( isOpenFun )
      {
        rDfr        <- data.frame(tokenStr, openNum, closeNum, firstNum )
        names(rDfr) <- names(retDfr)
        retDfr      <- rbind(retDfr, rDfr)
      }
    }
  }  
  retDfr
}
MtrSubFunDfr <- function(mt.list, funDfr, cmtDfr, funThatChr=NULL)
{
  funChr <- c("OrderSend", "OrderModify", "OrderClose", funThatChr)
  endNum <- length(mt.list)
  retDfr <- dataFrame( colClasses=c(Token="character", Open="numeric", 
                                    Close="numeric", First="numeric",
                                    Name="character"), nrow=0 )
  for( n in seq_along(funDfr$Open) )
  {
    #---  Knowledge when "for" has OrderSelect and ONE (1) or more of OrderSend
    #     (1) check for OrderSelect
    #     (2) check for OrderSend
    #     (3) check for OrderModify
    #     (4) check for OrderClose
    #     (5) check for OrderDelete
    tokenStr  <- funDfr$Token[n]
    openNum   <- funDfr$Open[n]
    closeNum  <- funDfr$Close[n]
    firstNum  <- funDfr$First[n]
    nameStr   <- funDfr$Name[n]
    rowNum    <- which(lapply(mt.list[openNum:closeNum],
                              function(x) { sum(grep("OrderSelect", x)) })>0)
    cmtBln    <- MtrIsComment("OrderSelect", rowNum+openNum-1, cmtDfr)
    isOpenSelect <- sum(cmtBln)<length(cmtBln)
    if( !isOpenSelect )
    {
      isOpenFun   <- FALSE
      for( o in seq_along(funChr) )
      {
        funStr    <- funChr[o]
        fRowNum   <- which(lapply(mt.list[openNum:closeNum],
                                  function(x) { sum(grep(funStr, x)) })>0)
        fCmtBln   <- MtrIsComment(funStr, fRowNum+openNum-1, cmtDfr)
        isOpenFun <- isOpenFun | (sum(fCmtBln)<length(fCmtBln))
      }
      if( isOpenFun )
      {
        rDfr        <- data.frame( tokenStr, openNum, closeNum, firstNum, nameStr )
        names(rDfr) <- names(retDfr)
        retDfr      <- rbind(retDfr, rDfr)
      }
    }
  }  
  retDfr
}
```

### 1.2.1) (size: ) In order to build the knowledge data sets, we create SIX (6)
functions: (i) MtrFindCmtDfr() returns a data frame of comments for both: (a)
single line comment, i.e. "//"; and (b) block comment, i.e. "/* */". There is NO
distinction between the TWO (2) types, except that a block comment has Close >
Open (but NOT always); (ii) MtrIsComment() returns a boolean vector given a
string for token, a numeric vector for rows, and a data frame for comments.
However, the algorithm is simple where ANY row is considered within a comment if
it lies between Open and Close. Todo: Some rows may contain both a comment and
valid tokens; (iii) MtrFindLoopDfr() returns a data frame of loops (default:
for), where a loop token is NOT valid if it is within a comment or a string.
Also, it has knowledge when OR NOT braces are used; (iv) MtrFindFunDfr() is a
copy of the previous function, with a slight modification. It validates a token
(public) by checking token+2 for "(", while the previous functions validates a
token (loop) by checking token+1 for "("; (v) MtrSubLoopDfr() returns a subset
of the data frame for loops by taking these steps: (a) check if there is a valid
token for query (OrderSelect) within the loop; (b) if yes, check if there is a
valid token for execution (OrderSend, OrderModify, OrderClose, OrderDelete)
within the loop. Hence, TWO (2) positives for both steps will result in a valid
loop. Note: this function allow you to specify additional execution tokens for
the second step; (vi) MtrSubFunDfr() returns a subset of the data frame for
functions by taking these steps: (a) check if there is NO valid token for query
(OrderSelect) within the function; (b) if no, check if there is a valid token
for execution within the function. Hence, a negative for the first step and a
positive for the second step will result in a valid function. Note: this subset
of the data frame for function (names) can be passed as the parameter of
additional execution tokens for the previous function MtrSubLoopDfr(). Todo:
There is no difference between execution tokens for function names and variable
names, e.g. OrderSelect() and TheseOrderSelect(), or OrderModify() and "int
OrderModifyBln".

```{.python .input}
%%R
cmtDfr  <- MtrFindCmtDfr(mt.list)
cmtDfr
forDfr  <- MtrFindLoopDfr(mt.list, cmtDfr, tokenChr=c("for", "while", "do"))
forDfr
funDfr  <- MtrFindFunDfr(mt.list, cmtDfr)
funDfr  <- funDfr[funDfr$Name!="init", ]
funDfr  <- funDfr[funDfr$Name!="start", ]
funDfr  <- funDfr[funDfr$Name!="deinit", ]
funDfr
fusDfr  <- MtrSubFunDfr(mt.list, funDfr, cmtDfr)
fusDfr
fu2Dfr  <- MtrSubFunDfr(mt.list, funDfr, cmtDfr, as.character(fusDfr$Name))
fu2Dfr
fosDfr  <- MtrSubLoopDfr(mt.list, forDfr, cmtDfr, as.character(fu2Dfr$Name))
fosDfr
sql.list <- list()
j <- 1
for( i in seq_along(mt.list) )
{
  #--- Read Line
  lineChr <- mt.list[[i]]
  
  #--- Account Query
  lineChr <- gsub("AccountFreeMargin", "GhostAccountFreeMargin", lineChr)
  
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
sql.list
```

### 1.3.1) (size: ) The internal function FuncRewrite() has an equivalent
function gsub() in R.
