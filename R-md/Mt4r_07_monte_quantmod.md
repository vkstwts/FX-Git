Mt4r_07_monte_quantmod
==================================================================================================================================
## Motivational Buckets
### 1.1) (size: )

```{.python .input}
%%R
if( Sys.info()["sysname"] == "Linux" )
  suppressPackageStartupMessages(source("~/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE))
if( Sys.info()["sysname"] == "Windows" )
  suppressPackageStartupMessages(source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE))
suppressPackageStartupMessages(source(paste0(RegRSourceDir(),"PlusMtrMonte.R"), echo=FALSE))
name.str    <- "Monte"
ver.str     <- "0.9.0"
mt4.str     <- "Go MT4 001/"
exe.dir     <- paste0(RegProgramDir(), mt4.str)
exe.str     <- "metalang.exe"
ea.dir      <- paste0(RegLocalProgramDir(), mt4.str, "experts/")
ind.dir     <- paste0(RegLocalProgramDir(), mt4.str, "experts/indicators/")
scr.dir     <- paste0(RegLocalProgramDir(), mt4.str, "experts/scripts/")
mqh.dir     <- paste0(RegLocalProgramDir(), mt4.str, "experts/include/")
MtrDeviceWriterStr(mqh.dir)
MtrMonteWriterStr(mqh.dir)
```

### 1.1.1) (size: )

```{.python .input}
%%R
bufNum        <- 1
styleChr      <- c('DRAW_NONE')
lType         <- c(rep('property', 3))
lName         <- c('indicator_buffers','indicator_color1','indicator_separate_window')
lVal          <- c(bufNum,'Black','')
eType         <- c('int','int','double')
eName         <- c('GdvPeriod','GdvLookBack','GdvAlpha')
eVal          <- c('20','500','1000.0')
mt.list       <- MtrAddRdeviceTop(name.str,ver.str,lType,lName,lVal,eType,eName,eVal)
mt.list       <- append( mt.list, MtrAddRdeviceInit(bufNum, Rsource="PlusMonte.R") )
mt.list       <- append( mt.list, MtrAddRStart() )
mt.list       <- append( mt.list, MtrAddRdeviceDeinit() )
mt.list       <- MtrAddInRmonte(mt.list)
mt.chr        <- MtrWriterStr(mt.list)
names(mt.chr) <- NULL
mt.chr
```

```{.python .input}
%%R
ea.dir.str  <- MtrEaWriterStr(name.str, mt.list, save.dir=ind.dir, ext.str="R.mq4")
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
