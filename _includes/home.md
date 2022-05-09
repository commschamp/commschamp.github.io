# What it's all about
**CommsChampion Ecosystem** is about **easy** and **compile-time configurable** implementation 
of binary communication protocols using C++11 programming language, 
with main focus on **embedded systems** (including bare-metal ones). 

Interested? Then buckle up and read on.

# Motivation
Almost every electronic device/component nowadays has to be able to communicate
to other devices, components, or outside world over some I/O link. Such communication
is implemented using various communication protocols, which are notorious for
requiring a significant amount of boilerplate code to be written. The implementation of
these protocols can be a tedious, time consuming and error-prone process.
Therefore, there is a growing tendency among developers to use third party code 
generators for data (de)serialization. Usually such tools receive description
of the protocol data structures in separate source file(s) with a custom grammar, 
and generate appropriate (de)serialization code and necessary abstractions to 
access the data. 

The main problem with the existing tools is that their major purpose
is data structures serialization and/or facilitation of remote procedure calls (RPC).
The binary data layout and how the transferred data is going to be used is of much
lesser importance. Such tools focus on speed of data serialization and I/O link transfer
**rather than** on safe handling of malformed data, compile time customization 
needed by various embedded systems and significantly reducing amount
of boilerplate code, which needs to be written to integrate generated
code into the product's code base. 

The binary communication protocols, which may serve as
an API or control interface of the device, on the other hand, require different approach.
Their specification puts major emphasis on binary data layout, what values are
being transferred (data units, scaling factor, special values, etc...) 
and how the other end is expected 
to behave on certain values (what values are considered to be valid and how to
behave on reception of invalid values). It requires having some extra meta-information
attached to described data structures, which is not sent over I/O link, but 
still needs to be accessible in the generated code. 
The existing tools either don't have means
to specify such meta-information (other than in comments), or don't know what to
do with it when provided. As the result the developer still has to write a 
significant amount of boilerplate code in order to integrate the generated 
serialization focused code to be used in binary communication protocol handling.
There is an article called 
[Communication is more than just serialization]({{ site.baseurl }}{% post_url 2019-01-05-communication-more-than-serialization %}) which provides more detailed examples on the mentioned
shortcomings.

All of the available schema based binary protocols generation solutions 
have multiple limitations,
such as inability to specify binary data layout or customize data structures 
used in the generated code. They also are unable to provide and/or customize
polymorphic interfaces to allow implementation of common
code that can work for all the message objects. Most available tools don't provide an
ability to customize and/or use multiple transport frames for the same messages,
but transferred over different I/O links. Few (if any) allow injection of
manually written code snippets in case the generated code is incomplete and/or
incorrect.

The generalization is hard. As the result many embedded C++ developers still have 
to manually implement required communication protocol 
rather than relying on the existing tools for code generation.

# Solution
**CommsChampion Ecosystem** is there it resolve all the problems listed above. 
Its main purpose is to allow quick and easy implementation of a custom binary
protocol, which may already been defined by a third party, without enforcing
any particular data encoding and/or layout. The main philosophy here is that
most communication protocols are very similar, **but** many may have a couple
of special nuances, which make it difficult to have "out of the box" complete
solution. The ecosystem allows injection of manually written code snippets, which
may replace and/or complement any functionality provided by default. 

There are multiple parts to the **CommsChampion Ecosystem**. 
Some are independent, some require other ones to be used as well.

- [COMMS library](https://github.com/commschamp/comms)
is the core component and the basis for the whole ecosystem. It is C++(11) **headers only**,
meta-programming friendly 
library that allows quick and easy implementation of binary communication protocols
using simple declarative statements of classes and types definitions.
These statements will specify **WHAT** needs to be implemented, 
the library internals handle the **HOW** part. The library contains multiple
"building blocks" that allow easy assembly of the protocol definition. It also
allows getting down to a single "brick" level when protocol definition and/or application
level customization is required.

- [CommsChampion Tools](https://github.com/commschamp/cc_tools_qt)
is a set of applications, which can be used to 
develop, monitor and debug custom binary communication protocols, that were
developed using the [COMMS library](https://github.com/commschamp/comms).
All the applications are plug-in based, i.e. plug-ins are used to define 
I/O socket, data filters, and the custom protocol itself. Such architecture allows
easy assemble of various protocol communication stacks. The tools are intended
to be used on **development PC**, and use [Qt5](http://www.qt.io/) framework 
for GUI interfaces as well as loading and managing plug-ins.

![tools image]({{ site.baseurl }}/image/cc_tools_qt.png){:style="width:650px"}

The [COMMS library](https://github.com/commschamp/comms) was
developed to allow quick and easy **manual** implementation of the protocol definition.
However, over the years it 
grew with features and accumulated multiple nuances to be remembered when defining 
a new protocol. In order to simplify protocol definition work, a separate toolset, called 
[commsdsl](https://github.com/commschamp/commsdsl), 
has been developed. It allows much easier and simpler definition of the protocol, 
using schema files written in XML based domain specific language, called 
[CommsDSL](https://github.com/commschamp/CommsDSL-Specification). The
[commsdsl](https://github.com/commschamp/commsdsl) repository provides the following
sub-components:

- **commsdsl2comms** application. It generates C++(11) code of the binary protocol 
definition out of [CommsDSL](https://github.com/commschamp/CommsDSL-Specification) 
schema file(s). The generated code is simple and easy to read. 
It uses [COMMS library](https://github.com/commschamp/comms)
to define all the necessary protocol definition classes and allows compile-time
application specific customization of polymorphic interfaces as well as selected
storage data structures. 

- **commsdsl2test** application. It generates C++(11) code that allows fuzz 
testing of the generated protocol code using 
[AFL](https://lcamtuf.coredump.cx/afl/) or similar. The 
generated code depends on the output generated by the **commsdsl2comms** as
well as [COMMS library](https://github.com/commschamp/comms).

- **commsdsl2tools_qt** application. It generates that implements a 
protocol definition plugin for the [CommsChampion Tools](https://github.com/commschamp/cc_tools_qt).
 The generated code depends on the output generated by the **commsdsl2comms** as
well as [CommsChampion Tools](https://github.com/commschamp/cc_tools_qt).

- **libcommsdsl** library. It provides a convenient interface to parse and
process [CommsDSL](https://github.com/commschamp/CommsDSL-Specification) schema
files. The library can be used to implement independent code generation tools,
which could be used to generate other independent code, such as bindings to the
code generated by the **commsdsl2comms** application for other programming
languages, extra testing, [wireshark](https://www.wireshark.org/) dissector, etc...

![commsdsl image]({{ site.baseurl }}/image/commsdsl.png){:style="width:868px"}

The process of the code generation is depicted below.

![generation image]({{ site.baseurl }}/image/generated.png){:style="width:988px"}

# Benefits

- No enforcing of particular data layout and/or encoding. Easy implementation of 
already defined third party protocol.

- Embedded (including bare-metal) friendly. The protocol definition code 
allows easy application specific **compile time** customization of polymorphic
interfaces and some data storage types. No usage of RTTI and/or exceptions. 
The usage of dynamic memory allocation may also be excluded.

- Robust and safe handling of malformed input data.

- Significantly minimizing amount of boilerplate code required to integrate the
usage of protocol definition into the business logic.

- Allowing injection of custom C++11 code snippets in case the code generated by the tools 
is incorrect or incomplete.

- Meta-programming friendly. Most classes define similar and predictable
public interface which allows easy compile time analysis and optimizations.

- Having "out of the box" protocol analysis and visualization tools.

- **NOT** using any socket / network abstractions and allowing full control
over serialized data for extra encryption, transport wrapping and/or external transmit over
some I/O link.

# Where to Start
At first, it is highly recommended to read through the 
available [tutorials](https://github.com/commschamp/cc_tutorial) which cover the most common 
aspects of the [CommsDSL](https://github.com/commschamp/CommsDSL-Specification), code generation,
[COMMS library](https://github.com/commschamp/comms) and their 
integration.

As the second stage in the learning process it is recommended to read through the 
[CommsDSL specification](https://commschamp.github.io/commsdsl_spec/) and 
[COMMS library documentation](https://commschamp.github.io/comms_doc).
The latter contains two major tutorial pages: 
- [How to Use Defined Custom Protocol](https://commschamp.github.io/comms_doc/page_use_prot.html) - 
  it explains how to customize the protocol definition for the end application needs and 
  how to integrate the protocol definition into the 
  application business logic.
- [How to Define New Custom Protocol](https://commschamp.github.io/comms_doc/page_define_prot.html) - 
  it explains how to define custom communication protocol
  from scratch and can be very useful when it comes to implementing custom code when the generated
  code is incomplete and/or incorrect.
  
The third stage is to review the available [examples]({{ site.baseurl }}/examples) and use 
them as reference and/or as a testing ground.

Other useful resources are:
- [commsdsl2comms Manual](https://github.com/commschamp/commsdsl/blob/master/doc/Manual_commsdsl2comms.md) - manual 
  of [commsdsl2comms](https://github.com/commschamp/commsdsl) utility, explains some command line 
  options and how to inject custom code snippets into the generated one.
- [Generated CMake Project Walkthrough](https://github.com/commschamp/commsdsl/blob/master/doc/GeneratedProjectWalkthrough.md) - 
  explains a directory structure of the code generated by the [commsdsl2comms](https://github.com/commschamp/commsdsl)
  application.
- [How to Use CommsChampion Tools](https://github.com/commschamp/comms_champion/wiki/How-to-Use-CommsChampion-Tools) - 
  explains how to use [CommsChampion Tools](https://github.com/commschamp/cc_tools_qt)
  to visualize and debug generated protocol code.
- [Testing Generated Protocol Code](https://github.com/commschamp/commsdsl/blob/master/doc/TestingGeneratedProtocolCode.md) - 
  explains how to fuzz test the protocol definition with [AFL](https://github.com/google/AFL).

# Communication
For any personal issue, question or request please send an 
[e-mail]({{ site.baseurl }}/contact/).

For any technical question that may be of interest to any other developer as
well please consider opening an issue on relevant project on [github](https://github.com/commschamp/).

It is also recommended to grab [RSS feed]({{ site.baseurl }}/feed.xml) of the 
[blog]({{ site.baseurl }}/blog/) 
to keep yourself updated with latest news and articles.

# New Features Development
The **CommsChampion Ecosystem** is already quite feature rich and satisfies the needs of most available
protocols. It's not in a constant active development. Introduction of new features now is on-request basis.
If you think that the latest [versions]({{ site.baseurl }}/versions) of the available components don't satisfy needs of your use case,
please [get in touch]({{ site.baseurl }}/contact/) to get an estimation of when the new functionality is going to be 
introduced.

# Contribution
For any open and freely available standard protocol you implement using the **CommsChampion Ecosystem**
please consider to open source your definition and implement it as a separate project, use
any of the available [examples]({{ site.baseurl }}/examples) as a template. Please also 
consider having it hosted as a project inside 
[CommsChampion Ecosystem github organization](https://github.com/commschamp). You'll be assigned as 
an owner and active maintainer of the project.
