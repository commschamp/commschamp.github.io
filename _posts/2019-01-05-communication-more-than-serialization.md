---
date: 2019-01-05
title: Communication is more than just serialization
categories:
  - articles
---
Every time the communication protocol is mentioned on any of the available
social resources, there always someone recommends to use
[ProtoBuf](https://developers.google.com/protocol-buffers/), 
[FlatBuffers](https://google.github.io/flatbuffers/),
[SimpleBinaryEncoding](https://github.com/real-logic/simple-binary-encoding),
or any other popular **serialization** tool or library. Most of these tools
were developed to facilitate remote procedure calls (RPCs) and in my 
opinion are not really suitable to be used as real communication protocols.
Those tools that were developed specifically for communication protocols still
focus mostly on fast data serialization and not on the proper "communication" part
described below.

They generate code that allows set / get of the values and maybe transfer raw 
data from one endpoint to another without any consideration on what exactly 
is being communicated to the other side and how the value is going to be used.
It leads to a significant amount of boilerplate code, which needs to be
written in order to integrate generated serialization code into the 
business logic. I'll give you an example.

Let's say you need to report value of some distance
between two points. You decide to serialize it as integer value and report
the distance in **centimeters**. It is not uncommon to develop a relevant business
logic (at least partially) that handles the protocol messages before the 
protocol itself was finalized and released. That's the nature of the development
process. Also let's assume your application uses floating point values and calculates
distances in **meters**. You take the received value, implement your math operations of 
units conversions and you are happy - everything works as normal. After a 
while you realize that **centimeters** don't give you enough precision for
some particular use case and you decide to change your protocol definition and 
report the distance in **millimeters** instead. If you use **ProtoBuf** (or similar) based
serialization solution there is no means to specify such change other than in comments.
You'll have to find all the places you used the old
value and manually update your math. I'd say it's not very efficient and
error-prone. If you are experienced developer you are
probably going to wrap such conversion math in a function, but still there is a need
to update the written business logic code in case the protocol definition changes.

----

The [CommsChampion Ecosystem]({{ site.baseurl }}) supports definition of value's units
in its DSL.
```
<int name="Distance" type="uint32" units="cm" />
```
The generated C++ code will contain the **compile time** meta information about the
units used and [COMMS library](https://github.com/arobenko/comms_champion#comms-library)
provides functions to calculate the value of needed units.
```cpp
auto distInMeters = comms::units::getMeters<double>(field_distance());
auto distInMillimeters = comms::units::getMillimeters<unsigned>(field_distance());
```
Even if units in the field definition change, the integration C++ code doesn't 
need to be changed. The recompilation of the source code will update the math.

There are some protocols that introduce some scaling (multiplication) factor and 
serialize floating point values as integers. 
The [CommsChampion Ecosystem]({{ site.baseurl }}) supports scaling as well.
For example, definition of latitude that multiplies floating point value
of degrees by 10'000'000 before serialization (and dividing after deserialization
to get the floating point value) may look like this:
```
<int name="lat" type="int32" units="deg" scaling="1/10000000" />
```
The integration code may use unit conversion functions provided by the 
[COMMS library](https://github.com/arobenko/comms_champion#comms-library)
to perform the right math.
```
auto latInDegrees = comms::units::getDegrees<double>(field_lat());
auto latInRadians = comms::units::getRadians<double>(field_lat());
``` 
All the math is done automatically and the C++ code doesn't need to be changed
when the scaling or units modified in the protocol definition DSL.

----

Another example would be to have values with special meaning. Let's say 
there is a need to communicate a delay in seconds before some event needs to 
happen. There should be some special value that indicates **infinite**
duration. Usually it is either **0** or maximum possible value of the
unsigned type being used. The developer that integrates the generated code into
the application needs to know what value it is. There is a need to write
at least one extra piece of boilerplate code that wraps the special value
and gives it a name. Wouldn't it be better if the generated code
contained such helper function already?

----

The DSL used by the [CommsChampion Ecosystem]({{ site.baseurl }}) allows
definition of the special values.
```
<int name="Timeout" type="uint32" units="sec">
    <special name="Infinite" val="0" />
</int>
```
The generated C++ code will contain the necessary set / get functions:
```cpp
struct Timeout : public comms::field::IntValue<...> 
{
    static constexpr std::uint32 valueInfinite() { return 0; }
    bool isInfinite() const { ...}
    void setInfinite() {...}
};
```
----


Also many communication protocols limit ranges of valid values for some
particular fields and require messages with invalid values being ignored as
malformed data. Usually serialization tools like **ProtoBuf** don't
provide such feature of specifying and validating the held value. Such valid
value ranges also have tendency to change (usually expand) from version to
version of the protocol. It also leads to a necessity to write extra boilerplate
code, that needs to be rechecked and modified every time the protocol is updated.

----

The DSL used by the [CommsChampion Ecosystem]({{ site.baseurl }}) allows
definition of multiple valid values and/or ranges.
```
<int name="SomeValue" type="uint8">
    <validRange value="[0, 10]" />
    <validValue value="15" />
    <validRange value="[53, 55]" />
</int>
```
The generated C++ code will contain the function to check the validity of the
field's value:
```cpp
class SomeValue : public comms::field::IntValue<...> 
{
public:
    bool valid() const { /* all the necessary checks here */ }
};
```
----

Such list of various communication nuances, which are not covered by pure
serialization and require extra boilerplate code, can go on and on. That's 
one of the reasons why the [CommsChampion Ecosystem](https://arobenko.github.io/cc/)
was developed. Another reason was to properly support embedded systems (which
in many cases cannot use the code generated by the tools like **ProtoBuf**), but
this is beyond the scope of this article.

Another observation worth mentioning is that many successful products and 
embedded devices in particular have open, simple, and clearly defined communication protocol
which can be used as a query / control interface to the product. Many such
embedded devices or sensors can be easily integrated into third party
solutions (such as various IOT control centers). In my opinion: openness, clearness,
and simplicity of the communication channel to the device is one of the 
reasons why the product becomes successful. Unfortunately many serialization 
tools or libraries focus on quick encoding / decoding of the serialized data
and their underlying protocol is far from being simple or clear, although open.
As the result, many products that use tools like **ProtoBuf** for their
communication channel have to provide additional SDKs to wrap the generated code
with easier to use API functions. That's extra development effort for the
product vendor and quite often the outcome still be not suitable for some
possible clients. My personal observation and my personal opinion is that _if
communication protocol is hidden behind some kind of SDK, it is usually a sign that
the protocol is poorly designed_.




