# SMDL

## Introduction

Nowadays there are plenty of libraries and protocols which address the problem of using abstract protocol and message descriptions to allow a descriptive pattern for protocol generation. Examples of this include Protobufs, Kaitai sctrucs, CAP-N-PROTO, UAVCAN DSDL and many others.

SMDL proposes that these libraries do a good job of solving the "machine" side of protocol definitions, but not such a good job at addressing the "semantic" problems. SMDL seeks to explore adding another layer to this stack, a "semantic" layer where the primary problem is how messages should be "interpreted" rather than "described"

### But why?

Lets imagine we have the following protocol buffers message defintion:

```
message ReadSensors {
  int mb_temperature = 0;
  int tp_temperature = 1;
  int mbus_voltage = 2;
  int mbus_current = 3;
}
```

This tells us everything we need to know to be able to serialise and deserialise the messages but the only clues we have on how to interpret these values is their names. There is also no way defined in the protobuf grammar to document the fields and message directly.

If we build the same message with Kaitai structs or UAVCAN DSDL we run int othe same problem each time, they are data structure description languages rather than languages to express the semantics of the messages. We can try and work around this in these languages
for example it is possible to use Doxygen with `.proto` files in order to embed documentation in them, this helps in allowing people to interpret the meaning of the message. With SMDL we are trying to add description of how to interpret a particular value into 
a regular format.

Lets add some psuedo information into the example message:
```
message ReadSensors {
  int mb_temperature = 0;
  mb_temperature.desc = "Temperature of the mainboard"
  mb_temperature.unit = "C";
  mb_temperature.transform = "y=(x*1024) + 23";

  int tp_temperature = 1;
  tp_temperature.desc = "Temperature of the top panel"
  tp_temperature.unit = "C";
  tp_temperature.transform = "y=(x*1024) + 23";

  int mbus_voltage = 2;
  mbus_voltage.desc = "Voltage of the main bus"
  mbus_voltage.unit = "V";
  mbus_voltage.transform = "y=(3.3/1024)*x";

  int mbus_current = 3;
  mbus_current.desc = "Current of the main bus"
  mbus_current.unit = "A";
  mbus_current.transform = "y=((3.3/1024)*x)/1000";
}
Readsensors.desc = "Message containing measurements from all onboard sensors"
```

We have added 3 attributes for each field of the message, `description`, `unit` and `transform`.

`description` gives us a formal way to document any particular element in the proto fule. `unit` allows us to give a particular unit to a value, such as volts or amps. Embedding this in the desctription allows the correct unit to be inferred when presenting information later.
Finally `transform` allows us to cater for the fact that the units used in a wire protocol at a low level rarely map 1 to 1, in these cases the "protocol" is transmitting raw ADC counts but now the extra information allows us to present SI units when the message is interpreted.

### Ok but what about messages that have meaning depending upon state?

Lets pretend that's not a thing for just now... the pattern can be extended to handle this by providing state objects and allowing the transforms to use elements of that state as values in their transormation..... but lets not try to address that quite yet.

### So what does SMDL actually do, and how far does it go?

Lets define a stack of layers similar to the OSI model but from a different perspective:

```
Semantic  
Transport  
Binary
```

What we are suggesting is that at the top there is a semantic layer, in the middle there is a transport layer and ultimately there is a binary layer which is the raw data of serialised messages.

If we look at how various protocol and data structure languages fit into this the we can see that they are defining a relationship between Transport and Binary. A protobufs definition maps descriptions
of transport objects to a raw binary representation. To go back to other formats such as UAVCAN DSDL and KAITAI struct. SMDL is intended to provide a mapping from semantics to transport objects.

### How can an abstract "transport object" be any use?

Referring once again to the various description langaueges we see that they all share one thing in common, each of them ultimately forms a graph. All of the formats allows messages
to contains other messages which means each description format ultimately provides a tree, where the root is the message definition and each leaf is a field in the message.

SMDL seeks to be applicable across various protocol libraries by referring to transport objects using `.` to access subfields. Lets go back to the original protobufs definition and provide an example
of how this will work.

Our original protobufs message was:

```
message ReadSensors {
  int mb_temperature = 0;
  int tp_temperature = 1;
  int mbus_voltage = 2;
  int mbus_current = 3;
}
```

A concept of an SMDL description which can map onto this is:
```
[
    "ReadSensors.mb_temperature" : {
        "description": "Temperature of the mainboard",
        "unit": "C",
        "transform": "y=(x*1024) + 23",
        "machine_repr": "int"
    }
]
```

With this added information where protobufs can automatically create serialisation code to get our message from one computer to another, the SMDL description can be used to format the contents of
the message in a way which is meaningful to humans. It should be observed that the .proto file could be derived from an SMDL description. It would be very desirable to make a message description language
which could in turn be fed down to the various protocol tools, such that perhaps a single semantic definition can transform messages across system boundaries consistantly. For example, imagine an embedded
system which communicates over CAN and then ultimately sends messages up to a server. It is desirable to use a protocol like UAVCAN on the canbus as it is designed for embedded systems and well constrained
which helps keep resource usage and error surface under control, but it is desirable to use a higher level protocol like protobufs in more powerful systems where flexibility is of more value than efficency.







