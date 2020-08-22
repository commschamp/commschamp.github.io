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
**DSL** schema definition as well as generated code. 

**NOTE**, that the generated code is actually a **CMake** project. Please
read the [Generated CMake Project Walkthrough](https://github.com/commschamp/commsdsl/wiki/Generated-CMake-Project-Walkthrough)
wiki page to understand what is being generated. Also note that the
generated protocol definition code is quite self-explanatory and how easy it is 
to read it. 

It is recommended to try and build the projects of the
[synthetic demo protocols]({{ site.baseurl }}/examples#synthetic-demo-protocols) while
enabling build of protocol plugin for 
[CommsChampion Tools](https://github.com/commschamp/comms_champion#commschampion-tools)
and follow instructions on 
[How to Use CommsChampion Tools](https://github.com/commschamp/comms_champion/wiki/How-to-Use-CommsChampion-Tools)
wiki page in order to get to know the environment for protocol visualization,
debugging and analysis.

----

The second stage is to learn how to use the generated code in some custom
application. Please download the doxygen generated documentation of the
[COMMS library](https://github.com/commschamp/comms_champion#comms-library) (named
**doc_comms_vX.zip**) from
the latest [release artifacts](https://github.com/commschamp/comms_champion/releases) 
and read **How to Use Defined Custom Protocol** tutorial page. It is also 
recommended to analyze the example client / server applications of the
[synthetic demo protocols]({{ site.baseurl }}/examples#synthetic-demo-protocols) and 
see how they follow the instructions from the tutorial. Note, that the
[synthetic demo protocols]({{ site.baseurl }}/examples#synthetic-demo-protocols) 
projects themselves has their own doxygen generated documentation in their
**release artifacts**. It may be beneficial the read those as well.

----

The third stage is to understand how to define new custom binary protocol. 
Please download and read the [CommsDSL](https://github.com/commschamp/CommsDSL-Specification/releases) 
specification first (read online 
[here](https://legacy.gitbook.com/book/arobenko/commsdsl-specification/details)).
It is recommended to use existing projects of 
[synthetic demo protocols]({{ site.baseurl }}/examples#synthetic-demo-protocols) as the basis
and introduce new fields / messages to their DSL shema(s). Please observe 
the generated code and/or review the changes in **cc_view** application
from [CommsChampion Tools](https://github.com/commschamp/comms_champion#commschampion-tools).

Read the [commsdsl2comms Manual](https://github.com/commschamp/commsdsl/wiki/commsdsl2comms-Manual)
to understand how to use **commsdsl2comms** code generator.

In case syntax of [CommsDSL](https://github.com/commschamp/CommsDSL-Specification) 
has some limitations that don't allow generation of the correct code and
injection of some custom code snippets are required, please 
do the following steps as well:
- Read the [commsdsl2comms Manual](https://github.com/commschamp/commsdsl/wiki/commsdsl2comms-Manual).
It explains what files need to be created and where to put them in order to
inject custom code into the generated one.
- Read **How to Define New Custom Protocol** tutorial page from the doxygen generated
documentation of the [COMMS library](https://github.com/commschamp/comms_champion#comms-library).
It explains what custom code is expected to look like.
- Review custom code snippets from available 
[real life open protocols]({{ site.baseurl }}/examples#real-life-open-protocols).

----

As the last stage it is recommended to fuzz test your protocol definition. 
Please read the 
[Testing Generated Protocol Code](https://github.com/commschamp/commsdsl/wiki/Testing-Generated-Protocol-Code) 
wiki page and use [AFL](http://lcamtuf.coredump.cx/afl/) to do so.

----

In case there are some open question remaining, please review the available tutorial
[articles]({{ site.baseurl }}/articles). Maybe your problem has already been solved and 
described in one of them. If not, please [send]({{ site.baseurl }}/contact) me a question. If 
your problem turns out to be common, it will find its way to one of the tutorial articles.
