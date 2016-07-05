---
title: Rethinking R Documentation
author: |
    Andrew Redd, PhD.  University of Utah  
    amredd+R@gmail.com  
    Andrew.Redd@hsc.utah.edu
date: UseR! 2016
theme: CambridgeUS
colortheme: beaver
header-includes:
    - \usepackage{framed}
    - \usepackage{FancyQuotes}
build-commands: |
  Rscript -e "knitr::knit('UseR2016 Presentation.Rmd')"
  pandoc -t beamer UseR2016\ Presentation.md -s --latex-engine=xelatex -o Presentation.pdf
...


Introduction
============

Documentation
-------------

\begin{columns}[T]
\begin{column}{0.30\textwidth}
\includegraphics[width=\textwidth]{CodeComplete.jpg}
\end{column}
\begin{column}{0.60\textwidth}
\begin{shadequote}{}
Many textbooks urge you to pile up a stack of information 
at the top of every routine, regardless of it's size or complexity.
This is ridiculous.
\end{shadequote}
\begin{shadequote}{Steve McConnell}
Follow the Principle of Proximity and put comments as close as 
possible to the code they describe.  They're more likely to be
maintained and they'll continue to be worthwhile.
\end{shadequote}
\end{column}
\end{columns}


Where we came from
------------------

* The Rd format
    - \LaTeX
    - Entirely Separate
    

Rd Format (load)
----------------

```r
\name{load}
\alias{load}
\title{Reload Saved Datasets}
\description{
  Reload the datasets written to a file with the function
  \code{save}.
}
\usage{
load(file, envir = parent.frame())
}
\arguments{
  \item{file}{a connection or a character string giving the
    name of the file to load.}
  \item{envir}{the environment where the data should be
    loaded.}
}
```

Rd Format (load)
----------------
```tex
\seealso{
  \code{\link{save}}.
}
\examples{
## save all data
save(list = ls(), file= "all.RData")

## restore the saved values to the current environment
load("all.RData")

## restore the saved values to the workspace
load("all.RData", .GlobalEnv)
}
\keyword{file}
```

Rd Format
---------

* Difficult to write
* Separated from the code
* Difficult to maintain



The Current State
=================

Roxygen
-------

* Puts documentation in same file as code
* special `#'` documentation comments
* No \LaTeX
* @tags

Roxygen Style
-------------
```r
#' @title Reload Saved Datasets
#' 
#' @description
#' Reload the datasets written to a file 
#' with the function \code{save}.
#' 
#' @param file a connection or a character string 
#'    giving the name of the file to load.
#' @param envir the environment where the data 
#'    should be loaded.
#' 
#' @seealso \code{\link{save}}.
#' ...
load <- function(file, envir=parent.frame(), verbose = FALSE)
{ 
    if(is.character(file)){
...
```

Remember this?
--------------

\begin{shadequote}{Steve McConnell}
Many textbooks urge you to pile up a stack of information 
at the top of every routine, regardless of it's size or complexity.
This is ridiculous.
\end{shadequote}

\center 

. . . 

Roxygen 

. . . 

\+ R coding requirements 

. . . 

= A stack of information at the top of every routine.

------

\center 
What else is there?

------

Doxygen
-------

> * The standard for C++ code documentation
> * Transforms directly into PDF or HTML
> * Adds relative documentation

Doxygen Examples - C++
----------------------

### Before
```cpp
/** This documents count **/
int count=0;
```

### After
```cpp
int count=0;//!< This also documents count
```

Doxygen
-------
Allows for documenting as close as possible
to the object as possible.



What can we do in R
===================


-----

\center \huge `lint`

\large Rethinking documentation


`lint`
------

### Goals for `lint`

> + Proximity
> + Inferential
> + Encapsulated
> + Associated
> + Modifiable


`lint`: methods
---------------

Use the R parser to take advantage of syntax knowledge.
This will enable;

> + **Proximity** 
>     - Put comments next to object they document
> + **Inference**
>     - What is being documented can be inferred, such as pulling names, from the code.    


`lint`: methods
---------------

We will store documentation as an R object.  Enabling:

> + **Encapsulated**
>     - A storage form for documentation besides Rd or HTML.
> + **Associated**
>     - Documentation stored as an attribute of a function.
> + **Modifiable**
>     - Enables dynamically generating documentation.
>     - Functions creating documented functions.


`lint`: new comments
--------------------

* **`#!`** The standard lint documentation tag.
* **`#!<`** Relative Tags, takes into account position.
* **`#!`**\textbf{\textasciicircum}  Continuation tags.

`lint`: Example : `load`
------------------------


```r
load <- 
function( file #!< a connection or a character string 
               #!^ giving the name of the file to load.
        , envir=parent.frame()
               #!< the environment where the data 
               #!^ should be loaded.
        )
{#! Reload Saved Datasets
 #!
 #! Reload the datasets written to a file 
 #! with the function \code{save}.
    if (is.character(file)) {
        con <- gzfile(file)
        on.exit(close(con))
        magic <- readChar(con, 5L, useBytes = TRUE)
        if (!length(magic)) 
            stop("empty (zero-byte) input file")
        if (!grepl("RD[AX]2\n", magic)) {
            if (grepl("RD[ABX][12]\r", magic)) 
                stop("input has been corrupted, with LF replaced by CR")
            warning(sprintf("file %s has magic number '%s'\n", 
                sQuote(basename(file)), gsub("[\n\r]*", "", magic)), 
                "  ", "Use of save versions prior to 2 is deprecated", 
                domain = NA, call. = FALSE)
            return(.Internal(load(file, envir)))
        }
    }
    else if (inherits(file, "connection")) {
        con <- if (inherits(file, "gzfile") || inherits(file, 
            "gzcon")) 
            file
        else gzcon(file)
    }
    else stop("bad 'file' argument")
    if (verbose) 
        cat("Loading objects:\n")
    .Internal(loadFromConn2(con, envir, verbose))
#' @seealso \code{\link{save}}.
}
```

`lint`: Example : `load`
------------------------

```r
load <- document_function(load)
load
```

```
## Reload Saved Datasets
## 
## # Description
## Reload the datasets written to a file 
## with the function \code{save}.
## 
## # Usage
## load(file, envir = parent.frame())
## 
## # Arguments
##     * `file`, 
##     * `envir`,
```

`lint`: Example : `load`
------------------------

```r
str(attr(load, 'docs'))
```

```
## List of 7
##  $ name       : chr "load"
##  $ title      : chr "Reload Saved Datasets"
##  $ description:Classes 'lint-description', 'lint-section', 'lint-documentation'  chr [1:2] "Reload the datasets written to a file " "with the function \\code{save}."
##  $ usage      : language load(file, envir = parent.frame())
##  $ arguments  :List of 2
##   ..$ file :List of 4
##   .. ..$ name       : chr "file"
##   .. ..$ description: chr ""
##   .. ..$ class      : atomic [1:1] ANY
##   .. .. ..- attr(*, "any.class")= logi TRUE
##   .. ..$ default    :Classes 'no-default-arg', 'argument-default', 'expression'   expression()
##   .. ..- attr(*, "class")= chr [1:3] "argument-info" "lint-documentation" "list"
##   ..$ envir:List of 4
##   .. ..$ name       : chr "envir"
##   .. ..$ description: chr ""
##   .. ..$ class      : atomic [1:1] ANY
##   .. .. ..- attr(*, "any.class")= logi TRUE
##   .. ..$ default    :Classes 'no-default-arg', 'argument-default', 'expression'   expression()
##   .. ..- attr(*, "class")= chr [1:3] "argument-info" "lint-documentation" "list"
##   ..- attr(*, "class")= chr [1:2] "arguments-list" "lint-documentation"
##  $ seealso    :Classes 'lint-seealso', 'lint-documentation'  chr "#' \\code{\\link{save}}."
##  $ sections   : Named list()
##  - attr(*, "class")= chr [1:3] "function-documentation" "lint-documentation" "list"
```


`lint`: Example : `load`
------------------------

```r
format_Rd(attr(load, 'docs')) %>%
    cat(sep='\n')
```

```
## \name{load}
## \title{Reload Saved Datasets}
## \description{
## 
## Reload the datasets written to a file with the function \code{save}.
## }
## \usage{load(file, envir = parent.frame())}
## \arguments{
## \item{file}{}
## \item{envir}{}
## }
## \seealso{#' \code{\link{save}}.}
```

Testing
-------

Put the tests in the same file

```r
if(FALSE){#! @testing
    # or @testthat
    # or @test
    expect_equal(1+1, 2)
}
```

Examples
--------

Same with examples

```r
if(F){#! @example
    hellO_world()
}
```

Summary
=======

Improvements
------------

* Documentation where it is most helpful.
* Most likely to be updated.
* Keeps all the good stuff.


Still to do
-----------

> * Incorporate into the prominent development tools
>     - `devtools`
>     - R Studio
> * Improve documenting non-functions.
>     - classes
>     - datasets
> * Get R-core to add inline comments
>     - Well, one can dream


Documentation Task Force
------------------------

![R Consortium Logo](rconsort_logo_sml.png)\

Yesterday The R Consotrium made a call for task force proposals.

. . .

**Documentation was mentioned specifically**

. . .

If you are interested takl to me <amredd+r@gmail.com>

Example Decision
----------------

I document this way:
```r
hello_world <- 
function( greeting  = 'hello' #< The greeting
        , recipient = 'world' #< Who to
        ){...}
```

. . .

most would write something like:

```r
hello_world <- 
function( greeting  = 'hello', #< The greeting
          recipient = 'world' #< Who to
        ){...}
```

. . .

Syntatically `#< the greeting` is grouped to the `recipient = 'world'` expression parrent.



