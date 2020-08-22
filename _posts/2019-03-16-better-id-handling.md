---
date: 2019-03-16
title: Better Custom Handling of Message ID
categories:
  - tutorials
---

This article describes new way (introduced in **v1.2** of the
[COMMS Library](https://github.com/commschamp/comms_champion#comms-library)
to implement custom handling of message ID information in transport framing.
It contains the same information as the
"Defining Custom Message ID Protocol Stack Layer" page from the 
[COMMS Library](https://github.com/commschamp/comms_champion#comms-library)
tutorial.

----

The [COMMS Library](https://github.com/commschamp/comms_champion#comms-library)
provides default **comms::protocol::MsgIdLayer** protocol 
stack layer to manage message ID information in the protocol framing. 
However, it may be insufficient (or incorrect) for some particular use cases, such as
using **bitfield** field to store both numeric message ID and some extra flags
[MQTT](http://mqtt.org) protocol does). 
The **Protocol Stack Definition Tutorial** page of the 
[COMMS Library](https://github.com/commschamp/comms_champion#comms-library) 
documentation explains how to define new (custom)
protocol layer. 

**NOTE**, that **comms::protocol::MsgIdLayer** class
contains significant amount of compile time logic of choosing the best
code, based on provided options as well as available polymorphic interface
messages being read and/or written. It is very impractical to try to 
implement something similar from scratch or copy-paste the existing 
definition and introduce required changes.

Since **v1.2** COMMS library provides an ability to extend the existing
definition of **comms::protocol::MsgIdLayer** and customize some bits
and pieces. Let's implement the mentioned example of sharing the same byte
for numeric ID and some flags.

First of all let's define the **Common Interface Class**, which
holds the **flags** information as data member of every message object.
In [CommsDSL](https://github.com/commschamp/CommsDSL-Specification) the definition
may look like this:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<schema name="my_prot" endian="big">
    <fields>
        <enum name="MsgId" type="uint8" semanticType="messageId" >
            <validValue name="Message1" val="1" />
            <validValue name="Message2" val="2" />
            <validValue name="Message3" val="3" />
            ...
        </enum>
    </fields>
    
    <interface name="Message">
        <set name="Flags">
            <bit name="bit0" idx="0" />
            <bit name="bit1" idx="1" />
            <bit name="bit2" idx="2" />
            <bit name="bit3" idx="3" />
        </set>
    </interface>
</schema>
```
The generated code (or manually implemented one) may look like one below
```cpp
namespace my_prot
{

// Enum used for numeric message IDs
enum MsgId
{
    MsgId_Message1,
    MsgId_Message2,
    ...
};

// Base class for all the fields defining serialization endian
using FieldBase = comms::field::Field<comms::option::BigEndian>;

// Definition of the message flags
class MessageFlags : public
    comms::field::BitmaskValue<
        FieldBase,
        comms::option::BitmaskReservedBits<0xf0>
    >
{
public:
    // Provides names and generates access functions for internal bits.
    COMMS_BITMASK_BITS_SEQ(bit0, bit1, bit2, bit3);
};

// Definition of the extensible common message interface
template <typename... TOptions>
class Message : public
    comms::Message<
        TOptions...,
        comms::option::BigEndian,
        comms::option::MsgIdType<MsgId>,
        comms::option::ExtraTransportFields<std::tuple<MessageFlags> >
    >
{
public:
    //  Allow access to extra transport fields.
    COMMS_MSG_TRANSPORT_FIELDS_ACCESS(flags);
};

} // namespace my_prot
```
Just to refresh the reader's memory: the usage of **COMMS_MSG_TRANSPORT_FIELDS_ACCESS()**
macro for the interface definition will generate **transportField_flags()**
convenience member function to access the stored **flags** field, while 
usage of **COMMS_BITMASK_BITS_SEQ()** in the flags field definition will
genereate **getBitValue_X()** and **setBitValue_X()** convenience member
functions to get / set values of the bits (where **X** is one of the defined
names: **bit0**, **bit1**, **bit2**, and **bit3**).

Now, let's define the **bitfield** field, that splits one byte in half to
store numeric message ID (in lower 4 bits) as well as extra flags (in upper 4 bits)
The [CommsDSL](https://github.com/commschamp/CommsDSL-Specification) definition is
```xml
<?xml version="1.0" encoding="UTF-8"?>
<schema name="my_prot" endian="big">
    <fields>
        <bitfield name="IdAndFlagsField">
            <int name="Id" type="uint8" bitLength="4" />            
            <int name="Flags" type="uint8" bitLength="4" />
        </bitfield>
    </fields>
</schema>
```
The generated code (or manually implemented one) should look similar to the 
code below
```cpp
namespace my_prot
{

class IdAndFlagsField : public
    comms::field::Bitfield<
        FieldBase,
        std::tuple<
            comms::field::IntValue<FieldBase, std::uint8_t, comms::option::FixedBitLength<4> >,
            comms::field::IntValue<FieldBase, std::uint8_t, comms::option::FixedBitLength<4> >
        >
    >
{
public:
    // Allow access to internal member fields.
    COMMS_FIELD_MEMBERS_ACCESS(id, flags);
};

} // my_prot
```
Again, just to refresh the reader's memory: the usage of **COMMS_FIELD_MEMBERS_ACCESS()**
macro for the bitfield definition will generate **field_X()**
convenience access member functions for the listed names.

Now it's time to actually extend the provided definition of 
the **comms::protocol::MsgIdLayer** and support usage of the defined
earlier **IdAndFlagsField** field. It must be written manually and injected
as extra code to **commsdsl2comms** code generator.
```cpp
namespace my_prot
{

namespace frame
{

namespace layer
{

template <
    typename TField, 
    typename TMessage, 
    typename TAllMessages, 
    typename TNextLayer, 
    typename... TOptions>
class MsgIdAndFlagsLayer : public
    comms::protocol::MsgIdLayer<
        TField,            // Will be IdAndFlagsField
        TMessage,          // Common interface class
        TAllMessages,      // std::tuple of all the supported input messages
        TNextLayer,        // Next layer in the protocol stack
        TOptions...,       // Application specific extension options
        comms::option::ExtendingClass<MsgIdAndFlagsLayer<TField, TMessage, TAllMessages, TNextLayer, TOptions...> >
                           // Make the comms::protocol::MsgIdLayer aware of it being extended
    >
{
    // Repeat definition of the base class
    using Base = comms::protocol::MsgIdLayer<...>;

public:
    // Repeat types defined in the base class (not visible by default)
    using MsgIdType = typename Base::MsgIdType;
    using MsgIdParamType = typename Base::MsgIdParamType;
    using Field = typename Base::Field; // same as IdAndFlagsField
    
    // Retrieve message ID value from the given IdAndFlagsField field
    static MsgIdType getMsgIdFromField(const Field& field)
    {
        return static_cast<MsgIdType>(field.field_id().value());
    }

    // Set flags value for the message object before proceeding to the next layer read
    // The message object is passed by reference
    template <typename TMsg>
    static void beforeRead(const Field& field, TMsg& msg)
    {
        msg.transportField_flags().value() = field.field_flags().value();
    }

    // Assemble the field's value before its write given message ID as well
    // as message object itself.
    template <typename TMsg>
    static void prepareFieldForWrite(MsgIdParamType id, const TMsg& msg, Field& field)
    {
        field.field_id().value() = static_cast<std::uint8_t>(id);
        field.field_flags().value() = msg.transportField_flags().value();
    }
};

} // namespace layer

} // namespace frame

} // namespace my_prot
```
The **comms::protocol::MsgIdLayer** doesn't have any virtual functions and
as the result not able to provide any polymorphic behavior. In order to be
able to extend its default functionality there is a need to use 
[Curiously Recurring Template Pattern](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern).
It is done by passing **comms::option::ExtendingClass** extension option with
the type of the layer class being defined to the **comms::protocol::MsgIdLayer**.

The extending class is expected to define the listed below functions. They do not
necessarily need to be **static**, accessing inner private state of the layer object
is also acceptable.

- **getMsgIdFromField()** - Member function that is invoked to retrieve the
numeric message ID out of the provided field object.
- **beforeRead()** - Member function that is invoked after appropriate
message object has been created but the read operation has **NOT** yet been
forwarded to the next protocol layer. It gives the developer a chance to
update some extra transport fields accessible via message interface class.
- **prepareFieldForWrite()** - Member function that is invoked to prepare
the field value before its write (serialization). After the function returns,
the **comms::protocol::MsgIdLayer** will invoke **write()** member function
of the passed field in order to serialize it.

The newly defined custom protocol stack layer can be used instead of
**comms::protocol::MsgIdLayer** when defining 
**protocol stack** (framing) of the protocol.

Just remember to still use **custom** layer when defining transport frame
using [CommsDSL](https://github.com/commschamp/CommsDSL-Specification) schema.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<schema name="my_prot" endian="big">
    <frame name="Frame">
        ...
        <custom name="IdAndFlags" idReplacement="true" field="IdAndFlagsField" />
        ...
    </frame> 
</schema>
```

