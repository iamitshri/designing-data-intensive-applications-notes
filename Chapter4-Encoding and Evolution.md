# Chapter 4: Encoding and Evolution

- In this chapter we will look at several formats for encoding data, including JSON, XML, Protocol Buffers, Thrift, and Avro. In particular, we will look at how they handle schema changes and how they support systems **where old and new data and code need to coexist**
  - Backward Compatibility: Newer code can read data that was written by older code
  - Forward Compatibility: Older code can read data that was written by newer code
- The translation from the in-memory representation to a byte sequence is called encoding (also known as serialization or marshalling), and the reverse is called decoding (parsing, deserialization, unmarshalling)

- It’s generally a bad idea to use your language’s built-in encoding for anything other than very transient purposes
  - Reasons: You are coupled to the language for long time, versioning data is an afterthought,encode decode performance etc 

- JSON, XML and Binary Variants
- Json,Xml
  - Widely adapted
  - Great for inter-organization communication
  - Json popular relative to xml. Xml is verbose and complicated. 
  - CSV is another language independent popular format
  - XML, CSV can't differenciate between number and string w/o external schema
  - JSON can distinguish between string and number but not between integer and floating-point numbers and doesn't specify precision
  - for example, integers greater than 2^53 cannot be exactly represented in an IEEE 754 double-precision floating-point number, so such numbers become inaccurate when parsed in a language that uses floating-point numbers
  - XML, JSON schema support is complicated to learn and support. CSV does not have schema, its upto application to decide the meaning of each row and column

  - As long as people agree on what the format is, it often doesn’t matter how pretty or efficient the format is. The difficulty of getting different organizations to agree on anything outweighs most other concerns.

  - Binary Encoding
   - For internal communication we could use compact and faster dataformat
   - Json less verbose than xml but still use a lot of space compared to binary formats
      - Binary encoding for JSON is available: MessagePack, BSON, UBJSON, BISON and Smile
      - XML binary encoding exampples: WBXML, Fast Infoset 
      - These formats have not been adopted a lot and not are not giving any significant space saving
  - Thrift and Protocol Buffers
    - Thrift developed at Facebook and Protocol Buffer at Google in 2007/08 
    - Both require schema
    - Thrift and Protocol Buffers each come with a code generation tool that takes a schema definition like the ones shown here, and produces classes that implement the schema in various programming languages 
    - Each field is identified by its tag number
    - You can change the name of a field in the schema, since the encoded data never refers to field names, but you cannot change a field’s tag, since that would make all existing encoded data invalid.
    - Forward Compatibility: Older code reading new field with new tag number simply ignores that field
    - The datatype annotation allows the parser to determine how many bytes it needs to skip. This maintains forward compatibility: old code can read records that were written by new code
    - What about backward compatibility? As long as each field has a unique tag number, new code can always read old data, because the tag numbers still have the same meaning. The only detail is that if you add a new field, you cannot make it required. If you were to add a field and make it required, that check would fail if new code read data written by old code, because the old code will not have written the new field that you added. Therefore, to maintain backward compatibility, every field you add after the initial deployment of the schema must be optional or have a default value

    - Avro
    - Apache Avro [20] is another binary encoding format that is interestingly different from Protocol Buffers and Thrift. It was started in 2009 as a subproject of Hadoop, as a result of Thrift not being a good fit for Hadoop’s use cases
    - Avro also uses a schema to specify the structure of the data being encoded
