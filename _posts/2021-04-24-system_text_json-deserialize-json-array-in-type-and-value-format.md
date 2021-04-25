---
layout: post
title: Deserialize JSON arrays from object type with $type and $values in System.Text.Json
date: 2021-04-24 00:00:00.000000000 -05:00
tags: [System.Text.Json deserializing JSON arrays with $type and $values]
description: Deserializing JSON arrays from object type with $type and $values in System.Text.Json
---
## Deserializing JSON arrays from object type with $type and $values in System.Text.Json

The [JSON spec](https://tools.ietf.org/html/rfc8259#section-2) says an array type property should be represented in a structure surrounded by square brackets.

 
>_An array structure is represented as square brackets surrounding zero
or more values (or elements).  Elements are separated by commas._

> _array = begin-array [ value *( value-separator value ) ] end-array_



Imagine working with a class like this for serialization/deserialization.

    public sealed class Person
    {
        public string FullName { get; set; }        
        public List<Vehicle> Vehicles { get; set; }
    }
    
    public sealed class Vehicle 
    {
        public int Year { get; set; }
        public string Model { get; set; }
    }

A valid JSON string adhering to RFC spec, which can be successfully deserialized to an instance of `Person` would be like this

    {
        "fullName": "Kramer",
        "vehicles": [
            {
                "year": 2012,
                "model": "Accord"
            },
            {
                "year": 2000,
                "model": "Altima"
            }
        ]
    }


Deserialization for the above JSON string works fine with both Newtonsoft JSON.NET and System.Text.JSON

    var personNewtonSoft = System.Text.Json.JsonSerializer.Deserialize<Person>(json, _options);
    var personSystemTextJson = Newtonsoft.Json.JsonConvert.DeserializeObject<CardInfo>(json);

## The other array format

I recently learned that arrays can be represented in another way in the JSON string. This is clearly not standard syntax as per the JSON spec. Take a look at this JSON string.

    {
        "fullName": "Kramer",
        "vehicles": {
        "$type": "System.Collections.Generic.List`1[[Vehicle, My.Project]], mscorlib",
        "$values": [
                {
                    "year": 2012,
                    "model": "Accord"
                },
                {
                    "year": 2000,
                    "model": "Altima"
                }
            ]
        }
    }
  

Here `vehicles` is using an object structure. The object has a `$type` and `$values` sub properties. The value part of  `$values` token actually holds the real array. [Newtonsoft.JSON.NET will successfully deserialize](https://www.newtonsoft.com/json/help/html/SerializeTypeNameHandling.htm) this JSON string to a `Person` instance while System.Text.Json Will throw.

> System.Text.Json.JsonException: The JSON value could not be converted to System.Collections.Generic.List`1[TestProject1.Vehicle]. Path: $.Vehicles | LineNumber: 2 | BytePositionInLine: 45.
    at System.Text.Json.ThrowHelper.ThrowJsonException_DeserializeUnableToConvertValue(Type propertyType)
   at System.Text.Json.JsonSerializer.HandleStartObject(JsonSerializerOptions options, ReadStack& state)
   at System.Text.Json.JsonSerializer.ReadCore(JsonSerializerOptions options, Utf8JsonReader& reader, ReadStack& readStack)
   at System.Text.Json.JsonSerializer.ReadCore(Type returnType, JsonSerializerOptions options, Utf8JsonReader& reader)
   at System.Text.Json.JsonSerializer.Deserialize(String json, Type returnType, JsonSerializerOptions options)
   at System.Text.Json.JsonSerializer.Deserialize[TValue](String json, JsonSerializerOptions options)

I was recently bit by this behavior while trying to migrate a system from JSON.NET to STJ. STJ is more strict and tries to adhere to the RFC spec, hence it consider this JSON string as non valid.

Ideally we should consider fixing the JSON to be following the spec. But in my case, this was not a straight forward option for the other team to change this right away. So I ended up writing a custom JsonConverter to do this.

A converter basically allows us to customize the behavior of serialization/deserialization. So in my case, I created a converter for handling the deserialization of the type `List<Vehicle>`

Take a minute to read the documentation: https://docs.microsoft.com/en-us/dotnet/standard/serialization/system-text-json-converters-how-to?pivots=dotnet-core-3-1

I ended up writing my converter like below. The trick is to loop through the tokens, when you find the StartArray type token, deserialize from that position onwards.

    public sealed class VehiclesArrayConverter : JsonConverter<List<Vehicle>>
    {
        public override List<Vehicle> Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
        {
            // Let's check we are dealing with a proper array format([]) adhering to the JSON spec.
            // JsonReader is a forward only reader.
            // We will read the next token from a copy of the JsonReader for checking our next token is a normal object.
            // If the first token is a normal object, we are dealing with a proper array.
            var readerCopy = reader;
            readerCopy.Read();
            if (readerCopy.TokenType == JsonTokenType.StartObject)
            {
                // Proper array, we can deserialize from this token onwards.
                return JsonSerializer.Deserialize<List<Vehicle>>(ref reader, options);
            }
            
            // If we reached here, it means we are dealing with the JSON array in non proper array form
            // ie: using an object structure with "$type" and "$values" format.
            // We will go through each token and when we get the array type inside (for $values),
            // We will deserialize that token. We exit when we reaches the next end object.

            List<Vehicle> list = null;
            while (reader.Read())
            {
                if (reader.TokenType == JsonTokenType.StartArray)
                {
                    list = JsonSerializer.Deserialize<List<Vehicle>>(ref reader, options);
                }
                if (reader.TokenType == JsonTokenType.EndObject)
                {
                    // finished processing the array and 
                    // reached the outer closing bracket token of wrapper object.
                    break;
                }
            }

            return list;
        }

        public override void Write(Utf8JsonWriter writer, List<Vehicle> value, JsonSerializerOptions options)
        {
            // No special handling. use default behavior
            JsonSerializer.Serialize(writer, value, value.GetType(), options);
        }
    }

Finally, you have to hook this up with the type/property. I chose to do this on the property level using the `JsonConverter` attribute.

    public sealed class Person
    {
        public string FullName { get; set; }
        
        [JsonConverter(typeof(VehiclesArrayConverter))]
        public List<Vehicle> Vehicles { get; set; }
    }

With this STJ can deserialize our non standard JSON string to `Person` instance properly.


![Test results](/assets/2021_04_24-STJ-JsonConverter-Tests.png)
Further reading:


* [https://github.com/dotnet/runtime/issues/30969#issuecomment-535779492](https://github.com/dotnet/runtime/issues/30969#issuecomment-535779492)

* [How to write custom converters for JSON serialization](https://docs.microsoft.com/en-us/dotnet/standard/serialization/system-text-json-converters-how-to?pivots=dotnet-core-3-1)

* [Utf8JsonReader](https://docs.microsoft.com/en-us/dotnet/api/system.text.json.utf8jsonreader?view=net-5.0)

* [JSON attacks](https://www.blackhat.com/docs/us-17/thursday/us-17-Munoz-Friday-The-13th-JSON-Attacks-wp.pdf)

* [https://blog.maartenballiauw.be/post/2020/01/29/deserializing-json-into-polymorphic-classes-with-systemtextjson.html](https://blog.maartenballiauw.be/post/2020/01/29/deserializing-json-into-polymorphic-classes-with-systemtextjson.html)(This was a really helpful post for me while writing the converter for the first time)


Cheers


