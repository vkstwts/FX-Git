Mt4r_02_deducer_rJava
==================================================================================================================================
## Motivational Buckets
### 1) (size: 52) This R Markdown file contains text and
code taken "verbatim" from Deducer's web site, "Working With Java Objects In R"
[0].

```{.python .input}
%%R
if( Sys.info()["sysname"] == "Linux" )
  source("~/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE)
if( Sys.info()["sysname"] == "Windows" )
{
  source("C:/Users/denbrige/100 FxOption/103 FxOptionVerBack/080 Fx Git/R-source/PlusReg.R", echo=FALSE)
  java.dir <- "C:\\Program Files\\Java\\jre7"
}
if( file.exists(java.dir) & Sys.getenv("JAVA_HOME")=="" )
  Sys.setenv(JAVA_HOME=java.dir)
suppressPackageStartupMessages(require( rJava ))
.jinit()
```

### 1.1.1) (size: ) We set the environment variable "JAVA_HOME" (JEV) to the
installation folder of the Java x64 Runtime Machine. Also, we will need to have
R x64 3.0.0. For x86 (32-bit), we set JEV to "C:\\Program Files
(x86)\\Java\\jre7" instead. This step is required for library(rJava) to load
correctly.
### 1.1.2) Similarly, before we can call any Java functions, we need
to initialize the Java Virtual Machine (JVM) with .jinit() function.

```{.python .input}
%%R
JDialog   <- J("javax.swing.JDialog")
myDialog  <- new(JDialog)
myDialog$setVisible(TRUE)
```

### 1.2.1) (size: ) Let's start by making a "JDialog" object. The first thing we
need to do is create a variable representing the class. This can be done with
the J() function, which takes as an argument the class location, and returns a
reference to that location (an S4 object of class "jclassName"). "JDialog" is
located in the "javax.swing" package, so we simply need to call. 
### 1.2.2) Now
that we have a reference to the "JDialog" class, we can a new object by calling
the new() function. new() takes as its first argument a "jclassName", e.g.
"JDialog", and any further arguments are passed to the "JDialog" constructor.
"myDialog" is now a reference (of S4 class "jobjRef") to an instance of
"JDialog".
### 1.2.3) We don't see anything yet, because we have not made the
dialog visible. There is a JDialog method called setVisible() which we can use
to make the dialog visible. rJava transparently takes care of the conversion
between R logical and Java boolean. We will go into conversions later on. You
should now see a small empty dialog window.

```{.python .input}
%%R
head(.jconstructors(JDialog))
head(.jmethods(myDialog))
head(.jfields(myDialog))
JDialog$isDefaultLookAndFeelDecorated()
JDialog$EXIT_ON_CLOSE
```

### 1.3.1) (size: ) We can list the available constructors with the
.jconstructors() function.
### 1.3.2) To look up what methods are available we
use the .jmethods() function.
### 1.3.3) We can also view the publicly
accessible fields of an object. 
### 1.3.4) Static methods can be called either
on the class or an object, though it is recommended that the class be used. 
###
1.3.5) Fields (variables specific to a particular Java class or object) can also
be accessed in a natural way.

```{.python .input}
%%R
myDialog$setTitle("cool dialog")
myDialog$setSize(200L,500L)
JLabel <- J("javax.swing.JLabel")
label <- new(JLabel,"Hi there, I'm a label")
myDialog$add(label)
myDialog$setVisible(TRUE)
myDialog$isVisible()
```

### 1.4.1) (size: ) You should now see a window 200 pixels wide and 500 pixels
long titled "cool dialog." We've also added a text label to the content of the
window. Notice that we used "200L" for the argument to setSize(). This is
because the "L" in R indicates that the number is an integer. Otherwise it would
be numeric, which would then be converted to a Java double which the method
doesn't understand.

```{.python .input}
%%R
myDialog %instanceof% J("java.awt.Dialog")
.jfloat(10)
.jarray(2)
```

### 1.5.1) (size: ) You may have noticed that we didn't actually pass any
"Frame" or "Dialog" objects to the constructor in the .jconstructors() example
above. Rather we gave it "JFrames" and "JDialogs" because they are subclasses of
"Frame" and "Dialog". We can check this using the %instanceof% operator. The
library(rJava) automatically takes care of the class conversions (called
casting) without any need for you to worry about it. Indeed, it works the other
way too. If a java method's signature says that it returns an object of class
Frame, but the object is actually a JFrame, it is automatically promoted to a
Frame and can be used as such. 
### 1.5.2) Some R data types can be
automatically converted to Java types when given to a method or constructor: (i)
numeric vector -> double[]; (ii) integer vector -> int[]; (iii) character vector
-> java.lang.String[]; (iv) logical vector -> boolean[]. For vectors of length
ONE (1), these will be converted to non-array types, e.g. integer -> int.
###
1.5.3) An R object can be converted to other basic Java data types with
.jfloat(), .jlong(), .jbyte(), .jchar() and .jshort(). If an R vector of length
ONE (1) needs to be passed as an array, simply wrap it in a .jarray() call. 

##
References
### 0) Deducer. Working With Java Objects In R. URL:
http://www.deducer.org/pmwiki/index.php?n=Main.WorkingWithJavaObjectsInR.
Accessed on 10 Apr 2013.
