---
date: 2019-06-08
title: Heterogeneous Lists
categories:
  - tutorials
---

This tutorial describes how to define and use heterogeneous lists in
[CommsChampion Ecosystem](https://commschamp.github.io).

----

Some protocols may require usage of heterogeneous fields or lists of 
heterogeneous fields, i.e. the ones that can be of multiple types. Good example 
would be a list of **properties**, where every property is a **key-value** pair 
or a **type-length-value** triplet (sometimes referred as **TLV**). The **key**
(or **type**) is usually a numeric ID of the property, 
while **value** can be any field of any length.

Let's start with an example of **key-value** pairs. At first there is a need
to define an appropriate heterogeneous field. It is done using **&lt;variant&gt;**
field definition.
```
<?xml version="1.0" encoding="UTF-8"?>
<schema name="my_prot" endian="big">
    <fields>
        <int name="PropKey" type="uint8" displayName="Key" failOnInvalid="true" displayReadOnly="true"/>
        
        <variant name="Property">
            <bundle name="Prop1">
                <int reuse="PropKey" name="Key" defaultValue="1" validValue="1" />
                <int name="Val" type="int16" />
            </bundle>
            <bundle name="Prop2">
                <int reuse="PropKey" name="Key" defaultValue="2" validValue="2" />
                <int name="Val" type="uint32" />
            </bundle>
            <bundle name="Prop3">
                <int reuse="PropKey" name="Key" defaultValue="3" validValue="3" />
                <string name="Val">
                    <lengthPrefix>
                        <int name="Length" type="uint8" />
                    </lengthPrefix>
                </string>
            </bundle>
        </variant>
    </fields>
</schema>
```

Please pay attention to the following details:

- The **&lt;variant&gt;** field has multiple **&lt;bundle&gt;** members and 
every **&lt;bundle&gt;** defines its first member to be a **Key**.
- Every **Key** reuses common definition of **PropKey** field defined earlier.
- Every **Key** has the same kind and underlying type (**&lt;int&gt;** and **uint8**)
- Every **Key** field sets its **validValue** and **defaultValue** properties
to have the same value.
- Every **Key** sets **failOnInvalid** property (copied from reused **PropKey** definition).

Now it is easy enough to put such a field into a list:
```
<?xml version="1.0" encoding="UTF-8"?>
<schema name="my_prot" endian="big">
    <fields>
        ...        
        <list name="PropsList" element="Property">
            <description>
                Properties list prefixed with total serialisation length
            </description>
            <lengthPrefix>
                <int name="Length" type="uint16" />
            </lengthPrefix>
        </list>
    </fields>
</schema>
```

The generated C++ code of the **&lt;variant&gt;** field defined above may look like this:
```cpp
class Property : public 
    comms::field::Variant<
        FieldBase,      // Base class of all the fields
        std::tuple<...> // Tuple of all the member bundles
    >
{
public:
     COMMS_VARIANT_MEMBERS_ACCESS(prop1, prop2, prop3);
};
```
The **COMMS_VARIANT_MEMBERS_ACCESS()** macro provided by the 
[COMMS library](https://github.com/commschamp/comms_champion#comms-library)
generates the following member type(s) and functions.
```cpp
class Property : public comms::field::Variant<...>
{
public:
    // Enumerator to access fields 
    enum FieldIdx {
        FieldIdx_prop1,
        FieldIdx_prop2,
        FieldIdx_prop3,
        FieldIdx_numOfValues
    }
    
    // Initialize internal storage as "prop1"
    template <typename... TArgs>
    auto initField_prop1(TArgs&&... args) -> decltype(initField<FieldIdx_prop1>(std::forward<TArgs>(args)...))
    {
        return initField<FieldIdx_prop1>(std::forward<TArgs>(args)...)
    }
    
    // Access internal storage already initialized as "prop1"
    auto accessField_prop1() -> decltype(accessField<FieldIdx_prop1>())
    {
        return accessField<FieldIdx_prop1>();
    }
    
    // Access internal storage already initialized as "prop1" (const variant)
    auto accessField_prop1() const -> decltype(accessField<FieldIdx_prop1>())
    {
        return accessField<FieldIdx_prop1>();
    }
    
    // Initialize internal storage as "prop2"
    template <typename... TArgs>
    auto initField_prop2(TArgs&&... args) -> decltype(initField<FieldIdx_prop2>(std::forward<TArgs>(args)...))
    {
        return initField<FieldIdx_prop2>(std::forward<TArgs>(args)...)
    }
    
    // Access internal storage already initialized as "prop2"
    auto accessField_prop2() -> decltype(accessField<FieldIdx_prop2>())
    {
        return accessField<FieldIdx_prop2>();
    }
    
    // Access internal storage already initialized as "prop2" (const variant)
    auto accessField_prop2() const -> decltype(accessField<FieldIdx_prop2>())
    {
        return accessField<FieldIdx_prop2>();
    }
    
    // Initialize internal storage as "prop3"
    template <typename... TArgs>
    auto initField_prop3(TArgs&&... args) -> decltype(initField<FieldIdx_prop3>(std::forward<TArgs>(args)...))
    {
        return initField<FieldIdx_prop3>(std::forward<TArgs>(args)...)
    }
    
    // Access internal storage already initialized as "prop3"
    auto accessField_prop3() -> decltype(accessField<FieldIdx_prop3>())
    {
        return accessField<FieldIdx_prop3>();
    }
    
    // Access internal storage already initialized as "prop3" (const variant)
    auto accessField_prop3() const -> decltype(accessField<FieldIdx_prop3>())
    {
        return accessField<FieldIdx_prop3>();
    }
};
```
 **NOTE**, that the provided names have propagated into definition of 
 **FieldIdx** enum as well as all **initField_X** and **accessField_X** functions.

When variant field object is instantiated, accessing the currently held field 
can be tricky though. There is a need to differentiate between **compile-time** 
and **run-time** knowledge of the contents.

When preparing a variant field (or message with variant fields) to be sent out, 
usually the inner field type and its value are known at **compile** time. 
The initialization of the field can be performed using one of the 
**initField_X()** member function described above:
```cpp
Property p; // Created in "invalid" state
auto& prop1 = p.initField_prop1(); // Initialise as Prop1 (constructor of prop1 is called)
...
```
or use inherited **comms::field::Variant::initField()** member function and generated 
**FieldIdx** enum as compile time access index:
```cpp
auto& prop1 = p.initField<Property::FieldIdx_prop1>();
```
The code snippets above provides a reference to the **Prop1** bundle field, definition
of which looks similar to the code below.
```cpp
class Prop1 : public 
    comms::field::Bundle<
        FieldBase,      // Base class of all the fields
        std::tuple<...> // Tuple of all the member bundles
    >
{
public:
     COMMS_FIELD_MEMBERS_ACCESS(key, val);
};
```
The **COMMS_FIELD_MEMBERS_ACCESS()** macro provided by the 
[COMMS library](https://github.com/commschamp/comms_champion#comms-library)
generates the following member type(s) and functions.
```cpp
class Prop1 : public 
    comms::field::Bundle<...>
{
public:
    // Access indices for member fields
    enum FieldIdx {
        FieldIdx_key,
        FieldIdx_value
    };

    // Accessor to "key" field
    auto field_key() -> decltype(std::get<FieldIdx_key>(value()))
    {
        return std::get<FieldIdx_key>(value());
    }

    // Accessor to const "key" field
    auto field_key() const -> decltype(std::get<FieldIdx_key>(value()))
    {
        return std::get<FieldIdx_key>(value());
    }

    // Accessor to "val" field
    auto field_val() -> decltype(std::get<FieldIdx_val>(value()))
    {
        return std::get<FieldIdx_val>(value());
    }

    // Accessor to const "val" field
    auto field_val() const -> decltype(std::get<FieldIdx_val>(value()))
    {
        return std::get<FieldIdx_val>(value());
    }
};
```
As the result, updating the value of the initialized property may look like this:
```cpp
prop1.field_val().value() = 0xff;
```
Note, that invocation of **.field_val()** provides a reference to the **&lt;int&gt;**
field (implemented as **comms::field::IntValue**) object and additional
invocation of **.value()** member function provides an access to the value
storage.

It is possible to re-initialize the field as something else, the previous 
definition will be properly destructed.
```cpp
Property p; // Created in "invalid" state
auto& prop1 = p.initField_prop1(); // Initialize as Prop1 (constructor of prop1 is called)
auto& prop2 = p.initField_prop2(); // Destruct Prop1 and initialize as Prop2
```
If the variant field has been initialized before, but there is a need to access 
the real type (also known at compile time), 
use appropriate **accessField_X()** member function:
```cpp
void updateProp1(Property& p)
{
    auto& prop1 = p.accessField_prop1(); // Access as Prop1 (simple cast, no call to the constructor)
    prop1.field_val().value() = 0xff; // Update the property value
}
```
or use inherited **comms::field::Variant::accessField()** member function and 
generated **FieldIdx** enum as compile time access index:
```cpp
auto& prop1 = p.accessField<Property::FieldIdx_prop1>();
```
There are cases (such as handling message object after "read" operation), when actual 
type of the **Property** field is known at **run-time**. The most straightforward 
way is to inquire the actual type index using **comms::field::Variant::currentField()**
function and then use a `switch` statement and handle every case accordingly.
```cpp
void handleProperty(const Property& p)
{
    switch(p.currentField())
    {
        case Property::FieldIdx_prop1:
        {
            auto& prop1 = p.accessField_prop1(); // cast to "prop1"
            ... // handle prop1;
            break;
        }
        
        case Property::FieldIdx_prop2:
        {
            auto& prop2 = p.accessField_prop2(); // cast to "prop2"
            ... // handle prop2;
            break;
        }
        ...
    };
}
```
However, such approach may require a significant amount of boilerplate code 
with manual (error-prone) "casting" to appropriate field type. 
The [COMMS library](https://github.com/commschamp/comms_champion#comms-library) 
provides a built-in way to perform relatively efficient (**O(log(n)**) way of 
dispatching the actual field to its appropriate handling function by using
**comms::field::Variant::currentFieldExec()** member function. It expects to 
receive a handling object which can handle all of the available inner types:
```cpp
struct PropertyHandler
{
    template <std::size_t TIdx>
    void operator()(Prop1& prop) {...}
    
    template <std::size_t TIdx>
    void operator()(Prop2& prop) {...}
    
    template <std::size_t TIdx>
    void operator()(Prop3& prop) {...}
}

void handleVariant(Property& p)
{
    p.currentFieldExec(PropertyHandler());
}
```
**NOTE**, that every `operator()` function receives a compile time index of the 
handed field within a containing tuple. If it's not needed when handling the 
member field, just ignore it or `static_assert` on its value if the index's 
value is known.

The class of the handling object may also receive the handled member type as a 
template parameter.
```cpp
struct PropertyHandler
{
    template <std::size_t TIdx, typename TField>
    void operator()(TField& prop) {...}
}
```
The example above covers basic **key-value** pairs type of properties. Quite 
often protocols use **type-length-value** (**TLV**) triplets instead. 
Adding length information allows having multiple value fields to follow 
(some of them may be introduced in future versions of protocols) as well as 
receiving unknown (to earlier versions of the protocol) properties and 
skipping over them.

Such triplets are properly supported in **v2** of 
[CommsDSL](https://github.com/commschamp/CommsDSL-Specification).
 The **key-value** definition above may be slightly altered to support such properties: 
```
<?xml version="1.0" encoding="UTF-8"?>
<schema name="my_prot" endian="big">
    <fields>
        <int name="PropType" type="uint8" displayName="Type" failOnInvalid="true" displayReadOnly="true"/>
        <int name="PropRemLen" type="uint16" displayName="Length" semanticType="length" displayReadOnly="true"/>
        
        <variant name="Property">
            <bundle name="Prop1">
                <int reuse="PropType" name="Key" defaultValue="1" validValue="1" />
                <ref field="PropRemLen" name="Length" />
                <int name="Val" type="int16" />
            </bundle>
            <bundle name="Prop2">
                <int reuse="PropType" name="Key" defaultValue="2" validValue="2" />
                <ref field="PropRemLen" name="Length" />
                <int name="Val" type="uint32" />
            </bundle>
            <bundle name="Prop3">
                <int reuse="PropType" name="Key" defaultValue="3" validValue="3" />
                <ref field="PropRemLen" name="Length" />
                <string name="Val" />
            </bundle>
            <bundle name="UnknownProp">
                <int reuse="PropType" name="Type" failOnInvalid="false" />
                <ref field="PropRemLen" name="Length" />
                <data name="Val" />
            </bundle>
        </variant>
    </fields>
</schema>
```

Please pay attention to the following details:

- Every **Length** references common definition of **PropRemLen** field defined earlier
using **&lt;ref&gt;** element.
- Every **Length** inherits **semanticType="length"** defenition from the
referenced field.
- The last element has non-failing read operation of the **Type** field, which
allows correct operation with unknown properties (which may be introduced
in the future versions of the protocol).

Such definition allows generation of correct code with correct handling of 
the remaining length done by the [COMMS library](https://github.com/commschamp/comms_champion#comms-library)
itself.

Also note, that the **Length** field specifies the **remaining length**
not including its own serialization length. In case the protocol specification
does demand to include the serialization length of the **Length** field itself,
it can be easily achieved by using **serOffset** property.
```
<?xml version="1.0" encoding="UTF-8"?>
<schema name="my_prot" endian="big">
    <fields>
        <int name="PropRemLen" type="uint16">
            <displayName value="Length" />
            <semanticType value="length" />
            <displayReadOnly value="true"/>
            <serOffset value="2" />
        </int> 
    </fields>
</schema>
```

The rest of the handling code presented above applies for this kind as well 
with one small nuance. The value of the **Length** field depends on the value of 
**Val** (especially with variable length fields like strings).

When such property field is default constructed, the **length** is updated to a 
correct value.
```cpp
Property p;
auto& prop1 = p.initField_prop1(); // Initialize as Prop1 (1 byte integral value)
assert(prop1.field_length().value() == 1U);

auto& prop3 = p.initField_prop3(); // Re-initialize as Prop3 (empty string)
assert(prop3.field_length().value() == 0U); // Remaining length of empty string
```

In case the **Val** member field of the **Prop3** gets updated, the value of 
**Length** field is not valid any more. There is a need to bring it into a 
consistent state by calling **refresh()** member function.
```cpp
prop3.field_val().value() = "hello";
prop3.refresh();
assert(prop3.field_length().value() == 5U);
```

Note, that there is no need to call **refresh()** after every update of a 
variant field. Usually such updates are done as preparation of the message 
to be sent. It is sufficient to call **doRefresh()** member function of the 
message object at the end of the update.
```cpp
SomeMessage msg;
auto& propsList = msg.field_propsList(); // access the properties list
auto& propsListVector = propsList.value(); // access the storage (vector);
propsListVector.resize(10); // create 10 properties (still invalid)

auto& prop1VariantField = propsListVector[0].initField_prop1(); // Initialize first as "prop1"
prop1VariantField.field_val().value() = 0xf;

auto& prop3VariantField = propsListVector[1].initField_prop3(); // Initialize second as "prop3"
prop3VariantField.field_val().value() = "hello";
...
msg.doRefresh(); // Bring all fields into a consistent state in one go
```
The [schema](https://github.com/commschamp/cc.demo1.commsdsl/blob/master/dsl/schema.xml) of
the [cc.demo1.commsdsl](https://github.com/commschamp/cc.demo1.commsdsl) example project
can be used for additional reference.
