# What it's all about
**CommsChampion Ecosystem** is about **easy** and **compile-time configurable** implementation 
of binary communication protocols using C++11 (and above) programming language,
with a main focus on **embedded systems** (including bare-metal ones).

Although the primary purpose of the **CommsChampion Ecosystem** is to help with the
implementation of the binary communication protocols, it can easily be used to encode /
decode any binary data as well.

Both **C++** and **embedded systems** are first class citizens of the **CommsChampion Ecosystem**.
However, other high level programming languages are also supported via generation of the
bindings (glue code) using [SWIG](https://www.swig.org/) and [emscripten](https://emscripten.org/).

Interested? Then buckle up and read on.

# Motivation
Almost every electronic device/component nowadays has to be able to communicate
with other devices, components, or the outside world over some I/O link. Such communication
is implemented using various communication protocols, which are notorious for
requiring a significant amount of boilerplate code to be written. The implementation of
these protocols can be tedious, time-consuming, and error-prone.
Therefore, there is a growing tendency among developers to use third-party code
generators for data (de)serialization. Usually, such tools receive descriptions
of the protocol data structures in separate source file(s) with custom grammar,
and generate appropriate (de)serialization code and necessary abstractions to
access the data.

The main problem with existing tools is that their major purpose
is data structure serialization and/or facilitation of remote procedure calls (RPC).
The binary data layout and how the transferred data is going to be used are of much
lesser importance. Such tools focus on the speed of data serialization and I/O link transfer
**rather than** on safe handling of malformed data, compile-time customization
needed by various embedded systems, and significantly reducing the amount
of boilerplate code, which needs to be written to integrate generated
code into the product's code base.

The binary communication protocols, which may serve as an API or control interface of the device,
on the other hand, require a different approach. Their specification puts major emphasis on binary
data layout, what values are being transferred (data units, scaling factor, special values, etc.),
and how the other end is expected to behave on certain values (what values are considered to be valid
and how to behave on reception of invalid values). It requires having some extra meta-information attached
to described data structures, which is not sent over the I/O link but still needs to be accessible in
the generated code. The existing tools either don’t have means to specify such meta-information
(other than in comments) or don’t know what to do with it when provided. As a result, the developer
still has to write a significant amount of boilerplate code in order to integrate the generated
serialization-focused code to be used in binary communication protocol handling.
There is an article called
[Communication is more than just serialization]({{ site.baseurl }}{% post_url 2019-01-05-communication-more-than-serialization %})
which provides more detailed examples of the mentioned shortcomings.

All available schema-based binary protocol generation solutions have
multiple limitations. These include an inability to specify binary data layout or
customize data structures used in the generated code. Additionally, they cannot
provide or customize polymorphic interfaces to allow the implementation of common
code that can work for all message objects. Most available tools lack an ability
to customize and/or use multiple transport frames for the same messages, but
transferred over different I/O links. Few, if any, allow the injection of manually
written code snippets in case the generated code is incomplete and/or incorrect.

The generalization is hard. As a result, many embedded C++ developers still have
to manually implement the required communication protocols rather than relying on
existing tools for code generation.

# Solution
**CommsChampion Ecosystem** is designed to address all the problems listed above.
Its main purpose is to allow quick and easy implementation of a custom binary
protocol, which may already have been defined by a third party, without enforcing
any particular data encoding and/or layout. The main philosophy here is that
most communication protocols are very similar, but many may have a couple
of special nuances, which make it difficult to have an "out of the box" complete
solution. The ecosystem allows injection of manually written code snippets, which
may replace and/or complement any functionality provided by default.

There are multiple parts to the **CommsChampion Ecosystem**. Some are independent,
while others require the use of additional components as well.

- [COMMS library](https://github.com/commschamp/comms) is the core component and
  the basis for the whole ecosystem. It is a C++(11) **headers only**, meta-programming
  friendly library that allows quick and easy implementation of binary communication
  protocols using simple declarative statements of classes and type definitions.
  These statements will specify **WHAT** needs to be implemented; the library internals
  handle the **HOW** part. The library contains multiple "building blocks" that allow
  easy assembly of the protocol definition. It also allows getting down to a single
  "brick" level when protocol definition and/or application-level customization is required.

- [CommsChampion Tools](https://github.com/commschamp/cc_tools_qt) is a set of applications
  that can be used to develop, monitor, and debug custom binary communication protocols
  that were developed using the [COMMS library](https://github.com/commschamp/comms).
  All the applications are plug-in based, meaning plug-ins are used to define I/O sockets,
  data filters, and the custom protocol itself. Such architecture allows easy assembly of
  various protocol communication stacks. The tools are intended to be used on a
  **development PC** and use the [Qt](http://www.qt.io/) framework for GUI interfaces
  as well as loading and managing plug-ins.

![tools image]({{ site.baseurl }}/image/cc_tools_qt.png){:style="width:650px"}

The [COMMS library](https://github.com/commschamp/comms) was developed to allow
quick and easy **manual** implementation of protocol definitions. However, over
the years, it has grown with features and accumulated multiple nuances to remember
when defining a new protocol. To simplify protocol definition work, a separate
toolset called [commsdsl](https://github.com/commschamp/commsdsl) has been developed.
It allows much easier and simpler definition of protocols using schema files written
in XML-based domain-specific language called
[CommsDSL](https://github.com/commschamp/CommsDSL-Specification).

The [commsdsl](https://github.com/commschamp/commsdsl) repository provides the
following sub-components:

- **commsdsl2comms** application generates C++(11) code of the binary protocol definition
  from [CommsDSL](https://github.com/commschamp/CommsDSL-Specification) schema file(s).
  The generated code is simple and easy to read. It utilizes the
  [COMMS library](https://github.com/commschamp/comms) to define all necessary protocol definition
  classes and allows compile-time application-specific customization of polymorphic interfaces
  and selected storage data structures, striking a balance between code clarity,
  boilerplate code generation, and code generated by the C++ compiler itself
  from the [COMMS library](https://github.com/commschamp/comms) internals.

- **commsdsl2test** application generates C++(11) code allowing fuzz testing of
  generated protocol code using [AFL](https://lcamtuf.coredump.cx/afl/) or similar.
  The generated code depends on the output of **commsdsl2comms** as well as
  [COMMS library](https://github.com/commschamp/comms).

- **commsdsl2tools_qt** application generates C++ code implementing a protocol
  definition plugin for [CommsChampion Tools](https://github.com/commschamp/cc_tools_qt).
  The generated code depends on the output of **commsdsl2comms** as well as
  [CommsChampion Tools](https://github.com/commschamp/cc_tools_qt).

- **commsdsl2swig** application generates [SWIG](https://www.swig.org/) interface
  files for creating bindings (glue code) to other high-level programming languages.
  The generated code depends on the output of **commsdsl2comms** and requires the
  usage of the external [swig](https://www.swig.org/) utility for bindings code generation.

- **commsdsl2emscripten** application generates class wrappers and
  [emscripten](https://emscripten.org/) bindings to protocol definition code generated by
  **commsdsl2comms**. The [emscripten](https://emscripten.org/) toolchain compiles
  the protocol definition C++ code into WebAssembly and generates JavaScript bindings.

- **commsdsl2c** application generates "C" interface
  for the C++ protocol definition code generated by the **commsdsl2comms**.
  It allows easier integration with other "C" code as well as other programming languages
  not covered by [SWIG](https://www.swig.org) or [emscripten](https://emscripten.org/).

- **commsdsl2latex** application generates LaTeX files for protocol specification.

- **libcommsdsl** library provides a convenient interface to parse and process
  [CommsDSL](https://github.com/commschamp/CommsDSL-Specification) schema files as
  well as facilitate the development of the new code generation applications.
  The library can implement independent code generation tools, used to generate
  other independent code such as bindings to code generated by **commsdsl2comms**
  for other programming languages, additional testing,
  [Wireshark](https://www.wireshark.org/) dissectors, etc.


![commsdsl image]({{ site.baseurl }}/image/commsdsl.png){:style="width:868px"}

The process of the code generation is depicted below.

![generation image]({{ site.baseurl }}/image/generated.png){:style="width:988px"}

# Benefits

- **No enforcement of particular data layout and/or encoding:** Allows easy
  implementation of already defined third-party protocols.

- **Embedded (including bare-metal) friendly:** The protocol definition code
  enables easy application-specific compile-time customization of polymorphic
  interfaces and some data storage types. It avoids the use of RTTI and/or
  exceptions and may exclude the usage of dynamic memory allocation.

- **Robust and safe handling of malformed input data.**

- **Significant reduction of boilerplate code:** Minimizes the amount of code
  required to integrate the protocol definition into the business logic.

- **Injection of custom C++11 code snippets:** Enables injection of custom code
  snippets in case the generated code is incorrect or incomplete.

- **Meta-programming friendly:** Most classes define a similar and predictable
  public interface, allowing for easy compile-time analysis and optimizations.

- **Out-of-the-box protocol analysis and visualization tools.**

- **No usage of socket/network abstractions:** Allows full control over serialized
  data for additional encryption, transport wrapping, and/or external transmission
  over some I/O link.

- **Support for other high-level programming languages via bindings (glue code).**

# Extra Libraries

The **CommsChampion Ecosystem** also provides several higher-level libraries for managing
some widely used open binary communication protocols. These libraries abstract away
the protocol messages and provide a convenient and easy-to-use API. All of the
provided libraries strive to adhere to the following principles:

- **Allow the end application to manage the I/O link connection and timers:**
  This allows the protocol handling library to be generic and platform-independent.

- **Ensure that the end application has full control over the exchanged raw data:**
  This enables the application to apply extra data manipulation, such as encryption,
  extra message framing when using non-reliable I/O link, and/or implement custom
  solutions extending the standard protocol functionality.

- **Provide a simple, asynchronous, and non-blocking interface.**

- **Offer an ability to customize the data structures and/or available functionality via compile-time configuration:**
  This makes it suitable for embedded systems (including bare-metal ones without heap).

- **Allow easy implementation of client/server applications**: Use any favorite event loop framework (such as Qt or Boost.Asio) or RTOS.

The list of currently available protocol handling libraries can be found on the [Projects]({{ site.baseurl }}/projects) page.

# More on Tools

[CommsChampion Tools](https://github.com/commschamp/cc_tools_qt) are highly versatile
applications that, once mastered, become indispensable for protocol development and testing.
They offer a wide range of functionalities, including:

- Sending and receiving messages to/from the tested system using various I/O links
  and additional transport protocols.

- Visualizing protocol messages.

- Decoding protocol messages from external raw data.

- Observing the exchange of protocol messages between nodes using "proxy" mode.

Even if the **CommsChampion Ecosystem** is not used to implement the required protocol
messages, with relatively little effort of defining the protocol
in terms of [CommsDSL](https://github.com/commschamp/CommsDSL-Specification) schema and then
using the provided code generators, the [CommsChampion Tools](https://github.com/commschamp/cc_tools_qt)
can become robust and valuable internal instrument of the defined protocol debugging.

# Mastering the CommsChampion Ecosystem
Please refer to the [Docs]({{ site.baseurl }}/docs) page for the available resources and
expected order of things to learn and practice.

# Asking for Help
Some aspects of the **CommsChampion Ecosystem** may be complex, overwhelming and/or
not explained properly in the available documentation. In case of difficulties it is recommended to fork
the [tutorial](https://github.com/commschamp/cc_tutorial) project on github and use
it as template to create special "tutorial" or "howto" folder (say howto100).
In that extra folder try to define minimal portion of the protocol that reproduces
the experienced problem. After that please send me an [e-mail]({{ site.baseurl }}/contact/)
containing the link to the forked project and the relevant details. I'll do my best to
review it and suggest a solution with the pull-request and/or explaining what needs
to change in comments.

# Other Communication
For any technical questions or feature requests that may be relevant to other developers,
please consider opening an issue on the relevant project's repository at [github](https://github.com/commschamp/).

For any other issue, question or request please send a direct
[e-mail]({{ site.baseurl }}/contact/).

It is also recommended to grab [RSS feed]({{ site.baseurl }}/feed.xml) of the 
[blog]({{ site.baseurl }}/blog/) 
to keep yourself updated with latest news.

# Contribution
For any open and freely available standard protocol you implement using the **CommsChampion Ecosystem**
please consider to open source your definition and implement it as a separate project, use
any of the already defined [protocols]({{ site.baseurl }}/projects) as a template. Please also
consider having it hosted as a project inside 
[CommsChampion Ecosystem github organization](https://github.com/commschamp). You'll be assigned as 
an owner and active maintainer of the project.
