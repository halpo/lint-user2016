---
title: Rethinking R Documentation
subtitle: an extension of the lint R package
author: Andrew Redd, PhD
...

In this presentation I will present an extension to the lint package to assist with documentation of R objects. R is the de facto standard for literate programming thanks to packages such as.  However, R still falls behind competing languages in the area of documentation.  In Steve McConnell's classic Code Complete (2004) his first principle of commenting routines is "keep comments close to the code they describe." The native documentation system for R requires separate files.  Packages have been developed that improve the situation, however unlike Doxygen, on which they are based, they do not allow full mixing code with documentation.

I propose a paradigm shift for R documentation, which I have implemented in the R package lint.  This strategy allows for several subtle changes in documentation, while seeking to preserve as much previous capability as is reasonable.  First is to store documentation as an R object itself, allowing for documentation to be dynamically generated and manipulated in code.  Documentation can also be kept as an attribute of the function or object that it documents and can exist independent from a package.  The second change is that the documentation engine makes full use of the R parser.  This integrates code with documentation comments and allows tailoring meaning to location.

These extensions give more capability to programmers and users of R easing the burden of creating documentation.  I welcome comments and discussion on the strategy of documentation and the direction of implementation.
