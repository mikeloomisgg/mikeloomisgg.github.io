---
layout: post
title: Lets Make a Serialization Library
comments: true
tags: [cpp]
---

Follow the code for this post on [github](https://github.com/mikeloomisgg/cppack).

Serialization is the most fundamental aspect of digital wares. It is what allows us to use all existing technology today, taking a real world application and enabling it to operate in a digital space. Computers work for us because we are able to translate our real world ideas and information into 1s and 0s.

Many times I have needed a serialization method for some engineering task. In languages like javascript, the language itself has built in facilicities for this, JavaScript Object Notation (JSON). JSON is a great serialization specification because it gives you a human readable representation of any arbitrary dynamically sized data types. Most of our internet communication uses JSON format today, even though no humans read it, and not all communication nodes are using JavaScript.

Binary serialization is another way we translate data, the difference is that the data stream is an unreadable stream of 1s and 0s. I'd like to use something like this for a database application I'm working on, and there are a couple of features that are important for me here:

- Easy to include - I'm going to be using this tool as an interface for a library. Therefore it needs to be stupid simple to use.
- Space efficient - The data output here should be as small as possible, and it doesn't need to be human readable.
- Portable - The data output should follow a spec capable of being platform agnostic. I don't care about endianness and it must work in any popular language.

so lets look at the most common tools we could use.

## Google Protobufs or Flatbuffers

[Protobufs](https://developers.google.com/protocol-buffers/) is popular, but I don't know why. As the website disclaims, you "use special generated source code to easily write and read your structured data". The concept of using generated code for this purpose is a true abomination. You define your data types in seperate .proto files, and a separate compiler spits out serialization functions for your language of choice.

Flatbuffer is similar, but you are capable of dereferencing individual data types from the stream without deserializing the entire object. This is a neat trick for some applications.

## Apache Avro
[Avro](https://en.wikipedia.org/wiki/Apache_Avro) is a slight improvement over protobufs; You don't have to compile the .schema files into generated code, they have generic generated code for your language that can just parse the .schema files.

This is what a "simple schema" file looks like. Yikes!

```json
{
   "namespace": "example.avro",
   "type": "record",
   "name": "User",
   "fields": [
      {"name": "name", "type": "string"},
      {"name": "favorite_number",  "type": ["int", "null"]},
      {"name": "favorite_color", "type": ["string", "null"]}
   ] 
 }
```

This style of serialization is just never going to work for my uses. I need to be able to define arbitrary objects in my code, and they might even have extra members or member functions that don't get serialized.

A perfect solution for my use would be a way to just flag a given struct or class as `Serializable`, and as Todd Howard says: "It just works".

### Cereal 

Cereal is a huge step in the right direction. Its stupid easy to use, you just add a single function to classes which you want to serialize. The fundamental types are automatically serialized thanks to static typing and template magic behind the scenes:

```c++
struct MyRecord
{
  uint8_t x, y;
  float z;
  
  template <class Archive>
  void serialize( Archive & ar )
  {
    ar( x, y, z );
  }
};

std::ofstream os("out.cereal", std::ios::binary);
  cereal::BinaryOutputArchive archive( os );

  MyRecord data;
  archive( data );
```

It supports portable binary output types as well as JSON. It can also be extended to support other serialization specs (like msgpack). Out of the box, I wouldn't be able to use its portable binary output, because it would be hard to unpack objects in other languages.

## Github Msgpack Libs

Msgpack is an open specification that allows any platform and language to serialize portable binary format data. This is exactly what I want, so lets look at some of the implementations in c++ on github.

### [msgpack-c](https://github.com/msgpack/msgpack-c)

This is the 'official' msgpack repo's c++ implementation:

```c++
#include <msgpack.hpp>
#include <string>
#include <iostream>
#include <sstream>

int main(void)
{
    msgpack::type::tuple<int, bool, std::string> src(1, true, "example");

    // serialize the object into the buffer.
    // any classes that implements write(const char*,size_t) can be a buffer.
    std::stringstream buffer;
    msgpack::pack(buffer, src);

    // send the buffer ...
    buffer.seekg(0);

    // deserialize the buffer into msgpack::object instance.
    std::string str(buffer.str());

    msgpack::object_handle oh =
        msgpack::unpack(str.data(), str.size());

    // deserialized object is valid during the msgpack::object_handle instance is alive.
    msgpack::object deserialized = oh.get();

    // msgpack::object supports ostream.
    std::cout << deserialized << std::endl;

    // convert msgpack::object instance into the original type.
    // if the type is mismatched, it throws msgpack::type_error exception.
    msgpack::type::tuple<int, bool, std::string> dst;
    deserialized.convert(dst);

    // or create the new instance
    msgpack::type::tuple<int, bool, std::string> dst2 =
        deserialized.as<msgpack::type::tuple<int, bool, std::string> >();

    return 0;
}
```

Cons:
- The serializer isn't using native types, its using special library types that mimic native types.
- I can't tell the library that an existing object of mine should be serializable, not without writing my own boilerplate glue code.
- The library throws runtime exceptions instead of returning error codes, something I cannot abide.

Therefore this library is unusable to me.

### [netLink](https://github.com/Lichtso/netLink)

Here is the example code for this implementation:
```c++
MsgPack::Serializer serializer(socket);  
std::vector<std::unique_ptr<MsgPack::Element>> arrayWithoutElements, arrayWith3Elements;
arrayWith3Elements.push_back(MsgPack::Factory(true));
arrayWith3Elements.push_back(MsgPack__Factory(Array(std::move(arrayWithoutElements))));
arrayWith3Elements.push_back(MsgPack::Factory("Hello World!"));  
serializer << MsgPack__Factory(Array(std::move(arrayWith3Elements)));

MsgPack::Deserializer deserializer(socket);  
deserializer.deserialize([](std::unique_ptr<MsgPack::Element> parsed) {
    std::cout << "Parsed: " << *parsed << "\n";
    return false;
}, true);
```

Pros/cons:
- The serializer requires a factory to turn native types into special library types. This is really bad.
- The library uses callbacks to report completion results of the deserialize operation. This is a step in the right direction for generic libraries that wish to support different styles of error handling.

### [goodform](https://github.com/jonathonl/goodform)

Here is the example code for this implementation:
```c++
std::stringstream ss;
goodform::any var, var2;
var = goodform::object
  {
    {"compact", true},
    {"schema", 0}
  };

goodform::msgpack::serialize(var, ss);
goodform::msgpack::deserialize(ss, var2);

goodform::form form(var2);

struct
{
  bool compact;
  std::int32_t schema;
} mpack;

mpack.compact = form.at("compact").boolean().val();
mpack.schema = form.at("schema").int32().val();

if (form.is_good())
{
  std::cout << "{ \"compact\": " << std::boolalpha << mpack.compact << ", \"schema\": " << mpack.schema << " }" << std::endl;
}
```

Pros/cons:
- The goodform::object is completely unusable for real applications.
- General structs must be explicitly serialized type by type.
- The error handling looks good, and hopefully doesn't throw any exceptions during failures.

So, its clear that none of the implementations in the world are appropriate. We need to copy how cereal manages to automatically and easily serialize objects, but we need to make it conform to the msgpack standard. I'm not going to attempt to extend the cereal library as they suggest is possible, because I don't understand the complex metaprogramming and [SFINAE](https://en.wikipedia.org/wiki/Substitution_failure_is_not_an_error) that they use to accomplish the type deduction.

# Get Started

I'm going to skip the part of setting up the build system, continuous integration, and test framework. You can check this [commit](https://github.com/mikeloomisgg/cppack/commit/c23fb709ac8821cf01f5c69a07365adf9dbdad3f) to see it.

I firmly believe in a test oriented design pattern. Let's define the way we want to interact with our library in an ideal way with test cases. This will shape the way that we proceed with developing the library:

```c++
struct Example {
  std::map<std::string, bool> map;

  template<class T>
  void msgpack(T &pack) {
    pack(map);
  }
};

TEST_CASE("Website example") {
  Example example{{{"compact", true}, {"schema", false}}};
  auto data = msgpack::pack(example);

  REQUIRE(data.size() == 18);
  REQUIRE(data == std::vector<uint8_t>{0x82, 0xa7, 0x63, 0x6f, 0x6d, 0x70, 0x61, 0x63, 0x74, 0xc3, 0xa6, 0x73, 0x63, 0x68, 0x65, 0x6d, 0x61, 0xc2});

  REQUIRE(example.map == msgpack::unpack<Example>(data).map);
}
```

There are 3 things that we provide users with to use the library.
1. A pack function which returns a byte array:  `std::vector<uint8_t> msgpack::pack(PackableStruct)`
2. An unpack function which returns the struct `PackableStruct msgpack::unpack<PackableStruct>(data)`
3. A template function that they add to structs they would like to serialize. This is what it means for a struct to be "Packable". This function also doubles as the method which deserializes data into the struct.

# Make it so

We can use c++ existing template system to do all of the type specific serialization operations. The entry point to all of our operations are a processing function:
```c++
template<class ... Types>
void process(Types &... args) {
    (pack_type(std::forward<Types &>(args)), ...);
}
```

First observation is that we are passing an arbitrary number of differently typed parameters to the function. So we use the c++11 [variadic](https://en.cppreference.com/w/cpp/language/parameter_pack) template feature. The process function now just accepts all the parameters that we pass to it.

Second, the arguments need to be individually serialized. We use a c++17 [fold](https://en.cppreference.com/w/cpp/language/fold) expression to do this.

At this point we implement a pack_type function:
```c++
template<class T>
void pack_type(const T &value) {
    if constexpr(is_map<T>::value) {
        pack_map(value);
    } else if constexpr (is_container<T>::value) {
        pack_array(value);
    } else {
        std::cerr << "Unknown type.\n";
    }
}

template<>
inline
void Packer::pack_type(const int8_t &value) {
  //**
}
```

If a type is given as a parameter, the compiler will match it to any specialized template functions before hitting the unspecialized version. So, a uint8_t parameter uses the bottom function and any other type uses the top function.

We also use the c++17 feature [if constexpr](https://en.cppreference.com/w/cpp/language/if) to deduce at compile time types that have nested values. This way we could serialize a array of `ints` just like we would serialize an array of `strings`. I believe if constexpr could have been used in the cereal library to avoid all the SFINAE type traits they used.

From this point on its just a lot of repitition to serialize and deserialize all of the supported types in the msgpack spec. Check out the repo to see how I did it.

# Problems with the spec

While making this library, I noticed a few things that would be useless to me about msgpack which I decided not to implement. This means that its possible to create msgpack objects which are valid to the spec but will fail to unpack when using this library.

## Null type
The null type in the spec seems pointless. A type which can only have 1 identity does not ever need to be serialized, so I don't know why it was included.

## The map
Msgpack spec defined a map as a name/value pair where the key is always a string and the value type can change between elements. This is only useful in a human readable map like you would see in something like JSON. The map I implemented is similar but it only operates as a map in c++ would.

## Endianness
The spec defines big endian for all of the types, but all of the implementations I looked at before making my own didn't actually attempt to ensure big endian serialization. Its a bit lazy, but I wanted my library to actually succeed in being portable across platforms so I did implement uniform endianness.

## Extension types and timestamps
I'm not convinced that the extensions feature is actually useful. Until I see a good reason, I'm just going to not support it. Any users with custom types could just as easily use the binary data type to implement their own serialization methods.

Likewise the timestamps are pointless. All possible timestamp uses are covered by the existing integer types.

# Thats it!

In closing, let me know if you used this library and if there are features you'd like to see.