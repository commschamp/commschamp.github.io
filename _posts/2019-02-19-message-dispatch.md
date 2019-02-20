---
date: 2019-02-19
title: Advanced Guide to Message Dispatching
categories:
  - tutorials
---

This article describes several new ways (introduced to **v1.1** of the
[COMMS Library](https://github.com/arobenko/comms_champion#comms-library))
to dispatch message object to its appropriate handling function.
It contains the same information as the
"Advanced Guide to Message Dispatching" page from the 
[COMMS Library](https://github.com/arobenko/comms_champion#comms-library)
tutorial.

----

# Existing Polymorphic Dispatch
Since the early days of the [COMMS Library](https://github.com/arobenko/comms_champion#comms-library)
it supported dispatching via polymorphic **dispatch()** member function of
the message object.
```
MsgPtr msg = ...; // std::unique_ptr to message object

msg->dispatch(handler); // Invoking appropriate handler.handle()  
```
Such dispatching is implemented using [Double Dispatch Idiom](https://en.wikipedia.org/wiki/Double_dispatch)
with **O(1)** runtime complexity. However, it requires the message interface
class to such polymorphic dispatch.
```
class MyHandler;

using MyMessage = 
    comms::Message<
        comms::option::Handler<MyHandler> // Provide handler class to implement dispatch()
        ...
    >;
```

# Additional Dispatch Methods for Message Object
Version **v1.1** of the [COMMS Library](https://github.com/arobenko/comms_champion#comms-library)
introduces several other ways to dispatch message object, which do not
require having polymorphic **dispatch()** member function in message interface.

The handler for the message object is expected to look exactly the same
as described in dispatching the message object using
[Existing Polymorphic Dispatch](#existing-polymorphic-dispatch), i.e. to define **handle()**
member function for every actual message type it intends to handle, 
**handle()** member function for the interface type for the ones it doesn't,
and define **RetType** type to specify return type of the handling functions
in case it's not **void**.
```
class MyHandler
{
public:
    // Return type of all the handle() functions
    typedef bool RetType;

    // All messages to handle properly
    bool handle(Message1<MyMessage>& msg) {...}
    bool handle(Message2<MyMessage>& msg) {...}
    bool handle(Message3<MyMessage>& msg) {...}
    ...
    // All other (don't care) messages
    bool handle(MyMessage& msg) {...}
};
```
There are several different implemented ways to dispatch a message object,
held by a pointer to its interface class, to its appropriate handling
function.
- [Polymorphic](#polymorphic-dispatch-of-message-object)
- [Static Binary Search](#static-binary-search-dispatch-of-message-object)
- [Linear Switch](#linear-switch-dispatch-of-message-object)

Every way has its advantages and disadvantages, please read on and choose
one that suites your needs. 
There are some definition commonly used for all the examples below.

All the mentioned below dispatch functions are defined in **comms/dispatch.h**
header.
```
#include "comms/dispatch.h"
```

The used name for the common interface class 
is going to be **MyMessage**
```
using MyMessage = comms::Message<...>;
```

The message types that need to be supported are bundled in
**std::tuple** and named **AllMessages**.
```
using AllMessages = 
    std::tuple<
        Message1<MyMessage>,
        Message2<MyMessage>,
        Message3<MyMessage>,
        ...
    >;
```
Also let's assume that numeric ID of **Message1** is **1**, of **Message2** is
**2**, of **Message90** is **90**, and so on...

## Polymorphic Dispatch of Message Object
The **polymorphic** dispatch of the message object can look like this
```
// Numeric ID of the message object
auto id = ...

// Message object itself held by a pointer to MyMessage interface class
MsgPtr msg = ...

// Handler object
MyHandler handler;

comms::dispatchMsgPolymorphic<AllMessages>(id, *msg, handler);
```
The **comms::dispatchMsgPolymorphic()** function will analyze the
provided tuple of message types (**AllMessages**) at **compile time**
and generate appropriate global static dispatch tables (initialized before
call to **main()**).

In case the numeric ID are sequential and unique with no more than 10% of the gaps
(the ID of the last message is not greater than number of message types in
the provided tuple multiplied by 1.1), the generated dispatch tables and logic provide
**O(1)** runtime complexity to dispatch message object into appropriate handler. 

The generated table is just an array of pointers to a dispatch method class
equivalent to the code below
```
struct DispatchMethod
{
    virtual RetType dispatch(MyMessage& msg, MyHandler& handler) const = 0;
};

// Dispatch registry
std::array<const DispatchMethod*, MaxId + 1> DispatchRegistry;
```
Every pointer to the array is to a global instantiation of the class below
for every type in the provided tuple.
```
<template <typename TMessage>
struct DispatchMethodImpl : public DispatchMethod
{
    virtual RetType dispatch(MyMessage& msg, MyHandler& handler) const override
    {
        return handler.handle(static_cast<TMessage&>(msg));
    }
};
```
The code inside the **comms::dispatchMsgPolymorphic()** function will
use the message ID as an index to access the registry array and invoke
the virtual **dispatch()** method. In case the accessed cell is empty, the
downcasting to the right message type won't occur and **handle()** message
for the interface of the handler object will be invoked.

In case the IDs of the message types in the provided tuple are too sparse,
The registry array will be packed (no holes inside) and binary search
using **std::lower_bound** algorithm is going to be performed. In this case
the **DispatchMethod** class will also report an ID of the message it is
responsible to handle via virtual function.
```
struct DispatchMethod
{
    virtual MsgIdParamType getId() const = 0;
    virtual RetType dispatch(MyMessage& msg, MyHandler& handler) const = 0;
};

<template <typename TMessage>
struct DispatchMethodImpl : public DispatchMethod
{
    virtual MsgIdParamType getId() const override
    {
        return TMessage::doGetId();
    }

    virtual RetType dispatch(MyMessage& msg, MyHandler& handler) const override
    {
        return handler.handle(static_cast<TMessage&>(msg);
    }
};
```
**NOTE**, that the performed binary search will invoke **O(log(n))** times
the virtual **getId()** member function to find the appropriate dispatch
method and then invoke virtual **dispatch()** one to downcast the message type
and invoke appropriate **handle()** member function of the handler.

There can also be a case when some message has multiple forms, that implemented
as different message classes, but which share the same ID. For example
```
using AllMessages = 
    std::tuple<
        Message1<MyMessage>, // Has ID 1
        Message2<MyMessage>, // Has ID 2
        Message90_1<MyMessage>, // Has ID 90
        Message90_2<MyMessage>, // Has ID 90
    >;
```
To support such case, the **comms::dispatchMsgPolymorphic()** function
is overloaded with new **offset** parameter (which is offset from the
first type in the tuple with the requested ID) to allow 
selection to what type to downcast.
```
comms::dispatchMsgPolymorphic<AllMessages>(90, msg, handler); // Invokes handle(Message90_1<MyMessage>&)
comms::dispatchMsgPolymorphic<AllMessages>(90, 0, msg, handler); // Invokes handle(Message90_1<MyMessage>&)
comms::dispatchMsgPolymorphic<AllMessages>(90, 1, msg, handler); // Invokes handle(Message90_2<MyMessage>&)
comms::dispatchMsgPolymorphic<AllMessages>(90, 2, msg, handler); // Out of range - invokes handle(MyMessage&)
```

There is also an overload to **comms::dispatchMsgPolymorphic()**, which 
doesn't receive any numeric message ID.
```
comms::dispatchMsgPolymorphic<AllMessages>(msg, handler);
```
Such call checks (at compile time) whether the message interface 
provides polymorphic dispatch.
If this is the case, then it is used to dispatch the message to the handler.
If not, then the message interface definition (**MyMessage**) 
must provide polymorphic ID value retrieval to be able to retrieve
ID of the message object. 

**SUMMARY**: The runtime complexity of **polymorphic** dispatch can be 
**O(1)** in case the numeric IDs of the supported message types in the provided tuple 
are **NOT** too sparse (no more than 10% holes). If this is not the case
the runtime complexity is **O(log(n))** with multiple virtual function calls
to retrieve the ID of the dispatching method. Also the downside of the
**polymorphic** dispatch is an amount of various v-tables the compiler will
have to generate, which can significantly increase the code size. It can
be a problem for various embedded systems with limited ROM.

## Static Binary Search Dispatch of Message Object
The **static binary search** dispatch of the message object can look like this
```
// Numeric ID of the message object
auto id = ...

// Message object itself held by a pointer to MyMessage interface class
MsgPtr msg = ...

// Handler object
MyHandler handler;

comms::dispatchMsgStaticBinSearch<AllMessages>(id, *msg, handler);
```
The **comms::dispatchMsgStaticBinSearch()** function generates the 
code equivalent to having the following folded if statements where **N**
is number of message types
```
if (id < id_of_elem(N/2)) {
    if (id < id_of_elem(N/4)) {
        ...
    else if (id > id_of_elem(N/4)) {
        ...
    }
    else {
        return handler.handle(static_cast<...>(msg)); // cast to appropriate type
    }
} else if (id > id_of_elem(N/2)) {
    if (id < id_of_elem(3N/4)) {
        ...
    else if (id > id_of_elem(3N/4)) {
        ...
    }
    else {
        return handler.handle(static_cast<...>(msg)); // cast to appropriate type
    }
}
else {
    return handler.handle(static_cast<...>(msg)); // cast to appropriate type
}
```
The runtime complexity of such code is always **O(log(n))** and there are no
extra v-tables and virtual functions involved.

In case there are distinct message types with the same numeric ID (multiple 
forms of the same message), the overloaded function with extra **offset**
parameter is provided (similar to described earlier **comms::dispatchMsgPolymorphic()**).
```
comms::dispatchMsgStaticBinSearch<AllMessages>(90, msg, handler); // Invokes handle(Message90_1<MyMessage>&)
comms::dispatchMsgStaticBinSearch<AllMessages>(90, 0, msg, handler); // Invokes handle(Message90_1<MyMessage>&)
comms::dispatchMsgStaticBinSearch<AllMessages>(90, 1, msg, handler); // Invokes handle(Message90_2<MyMessage>&)
comms::dispatchMsgStaticBinSearch<AllMessages>(90, 2, msg, handler); // Out of range - invokes handle(MyMessage&)
```

There is also an overload to **comms::dispatchMsgStaticBinSearch()**, which 
doesn't receive any numeric message ID.
```
comms::dispatchMsgStaticBinSearch<AllMessages>(msg, handler);
```
Such call requires the message interface definition (**MyMessage**) 
to provide **polymophic message ID retrieval** to be able to retrieve
ID of the message object. 

**SUMMARY**: The runtime complexity of the **static binary search** 
dispatch is always **O(log(n))** regardless of how sparse or compact are IDs
of the message types in the provided tuple. There are also no v-tables
generated by the compiler.

## Linear Switch Dispatch of Message Object
The **linear switch** dispatch of the message object can look like this
```
// Numeric ID of the message object
auto id = ...

// Message object itself held by a pointer to MyMessage interface class
MsgPtr msg = ...

// Handler object
MyHandler handler;

comms::dispatchMsgLinearSwitch<AllMessages>(id, *msg, handler);
```
The **comms::dispatchMsgLinearSwitch()** function generates the 
code equivalent to having the following folded switch statements.
```
switch (id) {
    case id_of_elem(0):
        return handler.handle(static_cast<...>(msg)); // cast to appropriate type
    default:
        switch(id) {
            case id_of_elem(1):
                return handler.handle(static_cast<...>(msg)); // cast to appropriate type         
            default:
                ...
        };
};
```
The runtime complexity of depends on the compiler being used. It has been
noticed that **clang** starting from **v3.9** generates efficient dispatch
table with **O(1)** complexity when binary code is optimized for speed (**-O2** or **-03**).
Other popular compilers, such as **gcc** and **MSVC** generate sequential 
comparison statements with **O(n)** runtime complexity.

In case there are distinct message types with the same numeric ID (multiple 
forms of the same message), the overloaded function with extra **offset**
parameter is provided (similar to described earlier **comms::dispatchMsgPolymorphic()**,
and **comms::dispatchMsgStaticBinSearch()**).
```
comms::dispatchMsgLinearSwitch<AllMessages>(90, msg, handler); // Invokes handle(Message90_1<MyMessage>&)
comms::dispatchMsgLinearSwitch<AllMessages>(90, 0, msg, handler); // Invokes handle(Message90_1<MyMessage>&)
comms::dispatchMsgLinearSwitch<AllMessages>(90, 1, msg, handler); // Invokes handle(Message90_2<MyMessage>&)
comms::dispatchMsgLinearSwitch<AllMessages>(90, 2, msg, handler); // Out of range - invokes handle(MyMessage&)
```
There is also an overload to **comms::dispatchMsgLinearSwitch()**, which 
doesn't receive any numeric message ID.
```
comms::dispatchMsgLinearSwitch<AllMessages>(msg, handler);
```
Such call requires the message interface definition (**MyMessage**) 
to provide **polymophic message ID retrieval** to be able to retrieve
ID of the message object. 

**SUMMARY**: The usage of **linear switch** dispatch is there for real
"stuntmen". If you are using **clang** compiler, able and willing to 
analyze generated binary code, and require optimal performance, then 
consider using **linear switch** dispatch. For all other cases its usage
is not recommended.

## Default Way to Dispatch Message Object
The [COMMS Library](https://github.com/arobenko/comms_champion#comms-library) 
also provides a default way to dispatch message object
without specifying type of the dispatch and allowing the library to choose
the best one by using **comms::dispatchMsg()**.
```
// Numeric ID of the message object
auto id = ...

// Message object itself held by a pointer to MyMessage interface class
MsgPtr msg = ...

// Handler object
MyHandler handler;

comms::dispatchMsg<AllMessages>(id, *msg, handler);
```
In such case the [COMMS Library](https://github.com/arobenko/comms_champion#comms-library) 
will check whether the condition of 
**O(1) polymorphic** dispatch tables holds true (no more than 10% holes in
the used IDs) and use **polymorphic** dispatch in this case. Otherwise
**static binary search** one will be used.

In case there are distinct message types with the same numeric ID (multiple 
forms of the same message), the overloaded function with extra **offset**
parameter is provided similar to other dispatch methods described above.
```
using AllMessages1 = 
    std::tuple<
        Message1<MyMessage>,
        Message2<MyMessage>,
        Message3<MyMessage>
    >;
comms::dispatchMsg<AllMessages1>(1, msg, handler); // Equivalent to using comms::dispatchMsgPolymorphic()

using AllMessages2 = 
    std::tuple<
        Message1<MyMessage>,
        Message2<MyMessage>,
        Message90_1<MyMessage>
    >;
comms::dispatchMsg<AllMessages2>(1, msg, handler); // Equivalent to using comms::dispatchMsgStaticBinSearch()
```

# Dispatch of the Message Type
In some occasions there is a need to know the exact message type given the
numeric ID without having any message object present for dispatching. The classic example
would be the creation of message object itself given the ID (that's what
**comms::MsgFactory** class does). To support such cases the 
[COMMS Library](https://github.com/arobenko/comms_champion#comms-library) 
provides the same 3 types of dispatching the given ID to its 
appropriate type.
- [Polymorphic](#polymorphic-dispatch-of-message-type)
- [Static Binary Search](#static-binary-search-dispatch-of-message-type)
- [Linear Switch](#linear-switch-dispatch-of-message-type)

For type dispatching the handler object is expected to look a bit 
different.
```
class MyHandler
{
public:
    template <typename TMessage>
    void handle() {...}
};
```
**NOTE**, that the actual type is passed to the **handle()** member function
as a template parameter. If some types require special handling function,
please use template specialization, like in the example below.
```
template <typename TMessage>
struct MyHandlerHelper
{
    // Generic handling function
    static void handle() {...}
};

template <>
struct MyHandlerHelper<my_protocol::Message1<MyMessage> >
{
    // Special handling function for my_protocol::Message1<MyMessage>
    static void handle() {...}
};

template <>
struct MyHandlerHelper<my_protocol::Message2<MyMessage> >
{
    // Special handling function for my_protocol::Message2<MyMessage>
    static void handle() {...}
};

class MyHandler
{
public:
    template <typename TMessage>
    void handle()
    {
        return MyHandlerHelper<TMessage>::handle();
    }
};
```
Similar to [Dispatch Methods for Message Object](#additional-dispatch-methods-for-message-object)
all the mentioned below dispatch functions are defined in **comms/dispatch.h**
header.
```
#include "comms/dispatch.h"
```

The message types that need to be supported are bundled in
**std::tuple** and named **AllMessages**.
```
using AllMessages = 
    std::tuple<
        Message1<MyMessage>,
        Message2<MyMessage>,
        Message3<MyMessage>,
        ...
    >;
```
All the type **dispatchMsgType*()** methods described below return 
**bool** which in case of being **true**
indicates that the type was successfully found and appropriate **handle()**
member function of the handler object being called. The return of **false**
indicates that the appropriate type hasn't been provided in **AllMessages**
tuple.

## Polymorphic Dispatch of Message Type
Just like with [Polymorphic Dispatch of Message Object](#polymorphic-dispatch-of-message-object), 
**polymorphic** dispatch of the message type generates 
similar dispatch tables with virtual functions for **O(1)** or **O(log(n))** runtime complexity
depending on how sparse the IDs in the provided tuple are.
```
MyHandler handler;
bool typeFound = dispatchMsgTypePolymorphic<AllMessages>(id, handler);
```
Please note, that in case there are distinct message types with the same numeric ID (multiple 
forms of the same message), the overloaded function with extra **offset**
parameter is also provided.
```
using AllMessages = 
    std::tuple<
        Message1<MyMessage>, // Has ID 1
        Message2<MyMessage>, // Has ID 2
        Message90_1<MyMessage>, // Has ID 90
        Message90_2<MyMessage>, // Has ID 90
    >;

dispatchMsgTypePolymorphic<AllMessages>(1, handler); // returns true
dispatchMsgTypePolymorphic<AllMessages>(1, 0, handler); // returns true, same as above
dispatchMsgTypePolymorphic<AllMessages>(1, 1, handler); // returns false
dispatchMsgTypePolymorphic<AllMessages>(90, handler); // returns true, handles Message90_1<MyMessage>
dispatchMsgTypePolymorphic<AllMessages>(90, 0, handler); // returns true, same as above
dispatchMsgTypePolymorphic<AllMessages>(90, 1, handler); // returns true, handles Message90_2<MyMessage>
dispatchMsgTypePolymorphic<AllMessages>(90, 2, handler); // returns false
```

## Static Binary Search Dispatch of Message Type
Similar to 
[Static Binary Search Dispatch of Message Object](#static-binary-search-dispatch-of-message-object), 
**static binary search** dispatch of the message type generates 
code equivalent to mentioned folded **if** statements with **O(log(n))** runtime
complexity.
```
MyHandler handler;
bool typeFound = dispatchMsgTypeStaticBinSearch<AllMessages>(id, handler);
```
Please note, that in case there are distinct message types with the same numeric ID (multiple 
forms of the same message), the overloaded function with extra **offset**
parameter is also provided.
```
using AllMessages = 
    std::tuple<
        Message1<MyMessage>, // Has ID 1
        Message2<MyMessage>, // Has ID 2
        Message90_1<MyMessage>, // Has ID 90
        Message90_2<MyMessage>, // Has ID 90
    >;

dispatchMsgTypeStaticBinSearch<AllMessages>(1, handler); // returns true
dispatchMsgTypeStaticBinSearch<AllMessages>(1, 0, handler); // returns true, same as above
dispatchMsgTypeStaticBinSearch<AllMessages>(1, 1, handler); // returns false
dispatchMsgTypeStaticBinSearch<AllMessages>(90, handler); // returns true, handles Message90_1<MyMessage>
dispatchMsgTypeStaticBinSearch<AllMessages>(90, 0, handler); // returns true, same as above
dispatchMsgTypeStaticBinSearch<AllMessages>(90, 1, handler); // returns true, handles Message90_2<MyMessage>
dispatchMsgTypeStaticBinSearch<AllMessages>(90, 2, handler); // returns false
```

## Linear Switch Dispatch of Message Type
Similar to 
[Linear Switch Dispatch of Message Object](#linear-switch-dispatch-of-message-object),
**linear switch** dispatch of the message type generates 
code equivalent to mentioned folded **switch** statements with **O(1)** runtime
complexity when compiled with **clang** compiler **v3.9** and above, and 
**O(n)** runtime complexity for other major compilers.
```
MyHandler handler;
bool typeFound = dispatchMsgTypeLinearSwitch<AllMessages>(id, handler);
```
Please note, that in case there are distinct message types with the same numeric ID (multiple 
forms of the same message), the overloaded function with extra **offset**
parameter is also provided.
```
using AllMessages = 
    std::tuple<
        Message1<MyMessage>, // Has ID 1
        Message2<MyMessage>, // Has ID 2
        Message90_1<MyMessage>, // Has ID 90
        Message90_2<MyMessage>, // Has ID 90
    >;

dispatchMsgTypeLinearSwitch<AllMessages>(1, handler); // returns true
dispatchMsgTypeLinearSwitch<AllMessages>(1, 0, handler); // returns true, same as above
dispatchMsgTypeLinearSwitch<AllMessages>(1, 1, handler); // returns false
dispatchMsgTypeLinearSwitch<AllMessages>(90, handler); // returns true, handles Message90_1<MyMessage>
dispatchMsgTypeLinearSwitch<AllMessages>(90, 0, handler); // returns true, same as above
dispatchMsgTypeLinearSwitch<AllMessages>(90, 1, handler); // returns true, handles Message90_2<MyMessage>
dispatchMsgTypeLinearSwitch<AllMessages>(90, 2, handler); // returns false
```

## Default Way to Dispatch Message Type
The [COMMS Library](https://github.com/arobenko/comms_champion#comms-library) 
also provides a default way to dispatch message type
without specifying type of the dispatch and allowing the library to choose
the best one using **comms::dispatchMsgType()** funtion.
```
// Numeric ID of the message object
auto id = ...

// Handler object
MyHandler handler;

comms::dispatchMsgType<AllMessages>(id, handler);
```
In such case the 
[COMMS Library](https://github.com/arobenko/comms_champion#comms-library) 
will check whether the condition of 
**O(1) polymorphic** dispatch tables holds true (no more than 10% holes in
the used IDs) and use **polymorphic** dispatch in this case. Otherwise
**static binary search** one will be used.

In case there are distinct message types with the same numeric ID (multiple 
forms of the same message), the overloaded function with extra **offset**
parameter is provided similar to other dispatch methods described above.
```
using AllMessages1 = 
    std::tuple<
        Message1<MyMessage>,
        Message2<MyMessage>,
        Message3<MyMessage>
    >;
comms::dispatchMsgType<AllMessages1>(1, handler); // Equivalent to using comms::dispatchMsgTypePolymorphic()

using AllMessages2 = 
    std::tuple<
        Message1<MyMessage>,
        Message2<MyMessage>,
        Message90_1<MyMessage>
    >;
comms::dispatchMsgType<AllMessages2>(1, msg, handler); // Equivalent to using comms::dispatchMsgTypeStaticBinSearch()
```

There are also **compile time** inquiry functions to check what type
of dispatch is going to be used for provided tuple of message types.
```
static_assert(comms::dispatchMsgTypeIsPolymorphic<AllMessages1>(), "Unexpected dispatch type");
static_assert(!comms::dispatchMsgTypeIsStaticBinSearch<AllMessages1>(), "Unexpected dispatch type");

static_assert(!comms::dispatchMsgTypeIsPolymorphic<AllMessages2>(), "Unexpected dispatch type");
static_assert(comms::dispatchMsgTypeIsStaticBinSearch<AllMessages2>(), "Unexpected dispatch type");
```


