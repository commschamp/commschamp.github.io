---
date: 2019-03-17
title: Better Custom Handling of Message Size
categories:
  - tutorials
---

This article describes new way (introduced in **v1.2** of the
[COMMS Library](https://github.com/arobenko/comms_champion#comms-library)
to implement custom handling of message size information in transport framing.
It contains the same information as the
"Defining Custom Message Size Protocol Stack Layer" page from the 
[COMMS Library](https://github.com/arobenko/comms_champion#comms-library)
tutorial.

----

The [COMMS Library](https://github.com/arobenko/comms_champion#comms-library) 
provides default **comms::protocol::MsgSizeLayer** protocol 
stack layer to handle remaining length information in the protocol framing. 
However, it may be insufficient (or incorrect) for some particular use cases, such as
using **bitfield** field to store both remaining size (length) and some extra flags. 
The **Protocol Stack Definition Tutorial** page of the 
[COMMS Library](https://github.com/arobenko/comms_champion#comms-library) 
documentation explains how to define new (custom)
protocol layer. 

However, since **v1.2** [COMMS Library](https://github.com/arobenko/comms_champion#comms-library) 
provides an ability to extend the existing
definition of **comms::protocol::MsgSizeLayer** and customize some bits
and pieces. Let's implement the mentioned example of sharing the same byte
for numeric ID and some flags.

For this example the protocol framing is defined to be
```cpp
ID | SIZE | PAYLOAD
```
**Note**, that **ID** layer, which is responsible to create proper message
object, precedes the **SIZE** one.

First of all let's define the **Common Interface Class**, which
holds the **flags** information as data member of every message object.
In [CommsDSL](https://github.com/arobenko/CommsDSL-Specification) the definition
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
names: **bit0**, **bit1**, **bit2, and **bit3**).

Now, let's define the **bitfield** field, that splits two bytes to
store remaining length (in lower 12 bits) as well as extra flags (in upper 4 bits).
The [CommsDSL](https://github.com/arobenko/CommsDSL-Specification) definition is
```xml
<?xml version="1.0" encoding="UTF-8"?>
<schema name="my_prot" endian="big">
    <fields>
        <bitfield name="SizeAndFlagsField">
            <int name="Size" type="uint16" bitLength="12" />            
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

class SizeAndFlagsField : public
    comms::field::Bitfield<
        FieldBase,
        std::tuple<
            comms::field::IntValue<FieldBase, std::uint16_t, comms::option::FixedBitLength<12> >,
            comms::field::IntValue<FieldBase, std::uint8_t, comms::option::FixedBitLength<4> >
        >
    >
{
public:
    // Allow access to internal member fields.
    COMMS_FIELD_MEMBERS_ACCESS(size, flags);
};

} // my_prot
```
Again, just to refresh the reader's memory: the usage of **COMMS_FIELD_MEMBERS_ACCESS()**
macro for the bitfield definition will generate **field_X()**
convenience access member functions for the listed names.

Now it's time to actually extend the provided definition of 
the **comms::protocol::MsgSizeLayer** and support usage of the defined
earlier **SizeAndFlagsField** field. It must be written manually and injected
as extra code to **commsdsl2comms** code generator.
```cpp
namespace my_prot
{

template <typename TField, typename TNextLayer, typename... TExtraOpts>
class MsgSizeAndFlagsLayer : public
    comms::protocol::MsgSizeLayer<
        TField,              // Will be SizeAndFlagsField
        TNextLayer,          // Next layer in the protocol stack
        TExtraOpts...        // Application specific extension options
        comms::option::ExtendingClass<MsgSizeAndFlagsLayer<TField, TNextLayer, TExtraOpts...> >
                             // Make the comms::protocol::MsgSizeLayer aware of it being extended
    >
{
    // Repeat definition of the base class
    using Base = comms::protocol::MsgSizeLayer<...>;

public:
    // Repeat types defined in the base class (not visible by default)
    using Field = typename Base::Field; // same as SizeAndFlagsField
    
    // Retrieve remaining length value from the given SizeAndFlagsField field
    static std::size_t getRemainingSizeFromField(const Field& field)
    {
        return static_cast<std::size_t>(field.field_size().value());
    }

    // Set flags value for the message object before proceeding to the next layer read
    // The message object is passed by pointer (which may be nullptr for some cases)
    template <typename TMsg>
    static void beforeRead(const Field& field, TMsg* msg)
    {
        assert(msg != nullptr); // mustn't be nullptr for this example
        msg->transportField_flags().value() = field.field_flags().value();
    }

    // Assemble the field's value before its write, given remaining length as well
    // as message object itself.
    template <typename TMsg>
    static void prepareFieldForWrite(std::size_t size, const TMsg* msg, Field& field)
    {
        auto& sizeMemberField = field.field_size();
        using SizeMemberFieldType = typename std::decay<decltype(sizeMemberField)>::type;
        sizeMemberField.value() = static_cast<typename SizeMemberFieldType::ValueType>(size);

        if (msg != nullptr) {
            field.field_flags().value() = msg->transportField_flags().value();
        }
    }
};

} // namespace my_prot
```
The **comms::protocol::MsgSizeLayer** doesn't have any virtual functions and
as the result not able to provide any polymorphic behavior. In order to be
able to extend its default functionality there is a need to use 
[Curiously Recurring Template Pattern](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern).
It is done by passing **comms::option::ExtendingClass** extension option with
the type of the layer class being defined to the **comms::protocol::MsgSizeLayer**.

The extending class is expected to define the listed below functions. They do not
necessarily need to be **static**, accessing inner private state of the layer object
is also acceptable.

- **getRemainingSizeFromField()** - Member function that is invoked to retrieve the
remaining length out of the provided field object.
- **beforeRead()** - Member function that is invoked before the read operation
is forwarded to the next layer. It gives the developer a chance to
update some extra transport fields accessible via message interface class (if 
such exists). Note that the message object is passed by the pointer to allow
cases when it is not created yet. In the example above **ID** layer precedes
the **SIZE**, so the message object must already be created.
- **prepareFieldForWrite()** - Member function that is invoked to prepare
the field value before its write (serialization). After the function returns,
the **comms::protocol::MsgSizeLayer** will invoke **write()** member function
of the passed field in order to serialize it. Note, that the message object is
passed by the pointer. There may be cases when **doUpdate()** member
function is going to be called without having actual message object being
present. In this case the **msg** parameter will be **nullptr**.

The newly defined custom protocol stack layer can be used instead of
**comms::protocol::MsgSizeLayer** when defining 
**protocol stack** (framing) of the protocol.

Just remember to still use **custom** layer when defining transport frame
using [CommsDSL](https://github.com/arobenko/CommsDSL-Specification) schema.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<schema name="my_prot" endian="big">
    <frame name="Frame">
        ...
        <custom name="MsgSizeAndFlagsLayer" field="SizeAndFlagsField" />
        ...
    </frame> 
</schema>
```


For completeness of the picture, let's also do similar example when
**SIZE** precedes the **ID**.
```cpp
SIZE | ID | PAYLOAD
```
The main problem with such scenario is that message object is created by
the **ID** layer, and is not available
in **beforeRead()** member function the **comms::protocol::MsgSizeLayer**
invokes. **However**, the flags may influence the way the message payload
is being processed, so the flags expected to be already assigned
to message object before message body is being read (before read operation
reaches **comms::protocol::MsgDataLayer**).

In order to resolve such case there is a need to do a bit of cheating by
introducing **pseudo** layer to manage **Extra Transport Values** after
the **ID** layer and before the **PAYLOAD**.
```cpp
SIZE | ID | FLAGS (pseudo) | PAYLOAD
```
The **SIZE** layer will access and assign the flags value to
**FLAGS (pseudo)** layer, which will reassign it to the message object created later
by the **ID** one. **NOTE**, that **pseudo transport value**
layer does **NOT** serialize its field and as the result preserves binary
compatibility of the protocol framing.

The extension to **comms::protocol::MsgSizeLayer** may be implemented like this:
```cpp
namespace my_prot
{

template <typename TField, typename TNextLayer, typename... TExtraOpts>
class MsgSizeAndFlagsLayer : public
    comms::protocol::MsgSizeLayer<
        TField,              // Will be SizeAndFlagsField
        TNextLayer,          // Next layer in the protocol stack
        TExtraOpts...        // Application specific extension options
        comms::option::ExtendingClass<MsgSizeAndFlagsLayer<TField, TNextLayer, TExtraOpts...> >
                             // Make the comms::protocol::MsgSizeLayer aware of it being extended
    >
{
    // Repeat definition of the base class
    using Base = comms::protocol::MsgSizeLayer<...>;

public:
    // Repeat types defined in the base class (not visible by default)
    using Field = typename Base::Field; // same as SizeAndFlagsField
    
    // Retrieve remaining length value from the given SizeAndFlagsField field
    static std::size_t getRemainingSizeFromField(const Field& field)
    {
        return static_cast<std::size_t>(field.field_size().value());
    }

    // Set flags value for the message object before proceeding to the next layer read
    template <typename TMsg>
    void beforeRead(const Field& field, TMsg* msg)
    {
        assert(msg == nullptr); // message is not created yet
        auto& pseudoFlagsLayer = Base::nextLayer().nextLayer(); 
        pseudoFlagsLayer.pseudoField().value() = field.field_flags().value();
    }

    // Assemble the field's value before its write, given remaining length as well
    // as message object itself.
    template <typename TMsg>
    static void prepareFieldForWrite(std::size_t size, const TMsg* msg, Field& field)
    {
        auto& sizeMemberField = field.field_size();
        using SizeMemberFieldType = typename std::decay<decltype(sizeMemberField)>::type;
        sizeMemberField.value() = static_cast<typename SizeMemberFieldType::ValueType>(size);

        if (msg != nullptr) {
            field.field_flags().value() = msg->transportField_flags().value();
        }
    }
};

} // namespace my_prot
```
The protocol stack (transport framing) itself needs to be defined like this:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<schema name="my_prot" endian="big">
    <frame name="Frame">
        <custom name="MsgSizeAndFlags" field="SizeAndFlagsField" />
        <id name="Id" field="MsgId" />
        <value name="Flags" interfaces="Message" interfaceFieldName="Flags" pseudo="true" field="SizeAndFlagsField" />
        <payload name="Data" />
    </frame> 
</schema>
```
The simplified version of the generated code may look similar to one below
```cpp
namespace my_prot
{

template <typename TMessage, typename TAllMessages>
struct Frame2 : public 
    MsgSizeAndFlagsLayer<                                        // SIZE + FLAGS 
        comms::protocol::MsgIdLayer<                             // ID
            comms::feild::EnumValue<FieldBase, MsgId>
            TMessage,
            TAllMessages,
            comms::protocol::TransportValueLayer<                // FLAGS
                comms::field::IntValue<FieldBase, std::uint8_t>, 
                0U,                                              
                comms::protocol::MsgDataLayer<>,                 // PAYLOAD      
                comms::option::PseudoValue                       // Make flags "pseudo" 
            >
        >
    >
{
    // Generate convenience access functions for various layers
    COMMS_PROTOCOL_LAYERS_ACCESS_OUTER(size, id, flags, payload);
};

} // namespace my_prot
```



