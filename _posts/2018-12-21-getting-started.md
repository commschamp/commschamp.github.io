---
date: 2018-12-21
title: Getting Started
categories:
  - tutorials
---

This article will guide your through several steps required to understand how to use
the **CommsChampion** ecosystems. It may take some time to master, but it's not 
really complicated. 

----

At first it is recommended to take a look at the definition of the [synthetic 
demo protocols]({{ site.baseurl }}/examples#synthetic-demo-protocols). Please review both the
**DSL** schema definition as well as generated code. Note how the generated
code is self-explanatory and how easy it is to read it. The headers-only
protocol definition files reside in **include** subdirectory.
The **cc_plugin** subdirectory contains code required to implement plugin for 
[CommsChampion Tools](https://github.com/arobenko/comms_champion#commschampion-tools).
Please ignore it while learning how to use the generated code.

**NOTE**, that the generated code is actually a **CMake** project. The main
*CMakeLists.txt* file lists available configuration options as well as
variables that can be used. In case the default values of the options are
not modified, such projects will build and install so called "full solution", 
which includes [CommsChampion Tools](https://github.com/arobenko/comms_champion#commschampion-tools)
and the protocol definition plugin to them in addition to headers-only 
[COMMS library](https://github.com/arobenko/comms_champion#comms-library) as
well the protocol definition headers themselves.

It is recommended to try and build the projects of the
[synthetic demo protocols]({{ site.baseurl }}/examples#synthetic-demo-protocols) and 
follow instructions on 
[How to Use CommsChampion Tools](https://github.com/arobenko/comms_champion/wiki/How-to-Use-CommsChampion-Tools)
wiki page in order to get to know the environment for protocol visualization,
debugging and analysis.

----

The second stage is to learn how to use the generated code in some custom
application. Please download the doxygen generated documentation of the
[COMMS library](https://github.com/arobenko/comms_champion#comms-library) (named
**doc_comms_vX.zip**) from
the latest [release artifacts](https://github.com/arobenko/comms_champion/releases) 
and read **How to Use Defined Custom Protocol** tutorial page. It is also 
recommended to analyze the example client / server applications of the
[synthetic demo protocols]({{ site.baseurl }}/examples#synthetic-demo-protocols) and 
see how they follow the instructions from the tutorial. Note, that the
[synthetic demo protocols]({{ site.baseurl }}/examples#synthetic-demo-protocols) 
projects themselves has their own doxygen generated documentation in their
**release artifacts**. It may be beneficial the read those as well.

----

The third stage is to understand how to define new custom binary protocol. 
Please download and read the [CommsDSL](https://github.com/arobenko/CommsDSL-Specification/releases) 
specification first (read online 
[here](https://legacy.gitbook.com/book/arobenko/commsdsl-specification/details)).
It is recommended to use existing projects of 
[synthetic demo protocols]({{ site.baseurl }}/examples#synthetic-demo-protocols) as the basis
and introduce new fields / messages to their DSL shema(s). Please observe 
the generated code and/or review the changes in **cc_view** application
from [CommsChampion Tools](https://github.com/arobenko/comms_champion#commschampion-tools).

In case syntax of [CommsDSL](https://github.com/arobenko/CommsDSL-Specification) 
has some limitations that don't allow generation of the correct code and
injection of some custom code snippets are required, please 
do the following steps as well:
- Read the [commsdsl2comms Manual](https://github.com/arobenko/commsdsl/wiki/commsdsl2comms-Manual).
It explains what files need to be created and where to put them in order to
inject custom code into the generated one.
- Read **How to Define New Custom Protocol** tutorial page from the doxygen generated
documentation of the [COMMS library](https://github.com/arobenko/comms_champion#comms-library).
It explains what custom code is expected to look like.
- Review custom code snippets from available 
[real life open protocols]({{ site.baseurl }}/examples#real-life-open-protocols).

----

In case there are some open question remaining, please review the available tutorial
[articles]({{ site.baseurl }}/articles). Maybe your problem has already been solved and 
described in one of them. If not, please [send]({{ site.baseurl }}/contact) me a question. If 
your problem turns out to be common, it will find its way to one of the tutorial articles.
