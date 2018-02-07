File_01_denbrige_dellm_testthat
==================================================================================================================================
## Motivational Buckets
### 1.1) (size: 26) We saved a snapshot image of Dell's
Mini Tower OEM Windows 7 Home Premium operating system (OS). Also, we installed
some desktop applications, such as R Studio, and some thin applications, such as
Git. Todo: (i) backup ".ssh" folder; (ii) backup "RescueMedia2013-04-05.iso"
file.
### 1.2) (size: 33) We explored BitBucket.org (BB) as a replacement for
GitHub.com (GH) due to the following reasons: (i) a public fork was created on
my Fx-Git repository; (ii) BB gives up to FIVE (5) private repositories for EACH
free account; (iii) BB has an option to prevent public forks of a repository.
Also, we saved a second snapshot (incremental) image of the OS. Note: For ATI:
(a) it is recommended to perform a disk cleanup before saving an image; and (b)
we excluded most folders in "C:/Users/denbrige" and files with extension
".vmdk".

```{.python .input}
%%R
if( Sys.info()["sysname"] == "Linux" )
  suppressPackageStartupMessages(source("~/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE))
if( Sys.info()["sysname"] == "Windows" )
  suppressPackageStartupMessages(source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE))
suppressPackageStartupMessages(source( paste0(RegGetRSourceDir(),"PlusFile.R") ))
suppressPackageStartupMessages(require( testthat ))
user.dir <- RegGetHomeDir()
thin.dir <- paste0(user.dir, "220 AppThin/222 AppThinVer/")
prg3.dir <- "C:/Program Files (x86)/"
prg6.dir <- "C:/Program Files/"
sys3.dir <- "C:/Windows/System32/"
expect_that( file.exists(paste0(prg3.dir,"Acronis/TrueImageHome/TrueImageLauncher.exe")), is_true() )
expect_that( !file.exists(paste0(user.dir,"Documents/RescueMedia2013-04-05.iso")), is_true() )
expect_that( file.exists(paste0(prg6.dir,"R/R-3.0.0/bin/x64/Rgui.exe")), is_true() )
expect_that( file.exists(paste0(prg6.dir,"RStudio/bin/rstudio.exe")), is_true() )
expect_that( file.exists(paste0(prg3.dir,"Mozilla Firefox/firefox.exe")), is_true() )
expect_that( file.exists(paste0(thin.dir,"001 Allway Sync 11.3")), is_true() )
expect_that( file.exists(paste0(thin.dir,"313 VLC Media Player 1.1.6 GPL")), is_true() )
expect_that( file.exists(paste0(thin.dir,"340 WinRar 4")), is_true() )
expect_that( file.exists(paste0(thin.dir,"454 USB Safely Remove")), is_true() )
expect_that( file.exists(paste0(thin.dir,"611 SmartGit 2.0")), is_true() )
expect_that( file.exists(paste0(thin.dir,"612 Notepad++")), is_true() )
expect_that( file.exists(paste0(thin.dir,"610 Git")), is_true() )
expect_that( file.exists(paste0(user.dir,".ssh/id_rsa")), is_true() )
expect_that( file.exists(paste0(user.dir,".ssh/id_rsa.pub")), is_true() )
expect_that( !file.exists(paste0(thin.dir,"399 TeamViewer 6")), is_true() )
```

### 1.1.1) (size: 26) We installed Acronis True Image Home 2012 Easter edition
(ATI) and created a 50 Gb Acronis Secure Zone (ASZ) (default: admin). We made a
full image backup, and set the subsequent images to incremental backups. Also,
we made a Rescue Media disc (iso) that NOT reside on the local hard disk. Note:
ATI has a sandbox installation feature.
### 1.1.2) We installed desktop
applications: (i) R 3.0.0; (ii) R Studio; (iii) Firefox. For the latter, we also
installed add-ons: (i) Memory Restart; (ii) Xmarks; (iii) LastPass; (iv) Video
DownloadHelper; (v) Download Statusbar; (vi) Memory Fox.
### 1.1.3) We installed
thin applications: (i) Allway Sync 11.3; (ii) VLC 1.1.6; (iii) WinRar 4; (iv)
USB Safely Remove; (v) SmartGit 2.0; (vi) Notepad++; (vii) Git. For the latter,
we copied the files "id_rsa" and "id_rsa.pub" from the older Sony laptop. Also,
we did NOT install TeamViewer 6, because the newer versions (8+) are NOT
backward compatible.

```{.python .input}
%%R
expect_that( file.exists(paste0(prg3.dir,"VMware/VMware Workstation/vmware.exe")), is_true() )
expect_that( file.exists(paste0(prg3.dir,"Canon/MP Navigator EX 3.0/mpnex30.exe")), is_true() )
expect_that( file.exists(paste0(sys3.dir,"spool/drivers/x64/3/fpdisp6.exe")), is_true() )
expect_that( file.exists(paste0(sys3.dir,"spool/drivers/x64/3/fppdis3a.exe")), is_true() )
expect_that( file.exists(paste0(sys3.dir,"CanonIJ Uninstaller Information")), is_true() )
expect_that( file.exists(paste0(user.dir,"My Virtual Machines/910d Vista SP2 ThinApp 4.7")), is_true() )
expect_that( file.exists(paste0(thin.dir,"611b SmartGit 4.0.5/SmartGitHg 4.exe")), is_true() )
```

### 1.2.1) (size: 33) We installed desktop applications: (i) VMware Workstation
8.0 (VW8) from folder "003 VMware" in external hard disk "Mybookfive" (default:
admin), using a legit setup file, but an illegal key
"RU4L5-88745-X8L2H-MFFZ2-KZZ8N"; (ii) Canon MP Navigator 3.05 for Windows 7
(default: admin); (iii) FinePrint 6.25 for Windows 7 (default: admin); (iv)
pdfFactory Pro 3.53 
### 1.2.2) We installed peripheral drivers: (a) Canon MP640
(MP648) series driver 1.05 for Windows XP (x64).
### 1.2.3) We copied a virtual
machine "910d Vista SP2 ThinApp 4.7", and created a thin application "611b
SmartGit 4.0.5".
