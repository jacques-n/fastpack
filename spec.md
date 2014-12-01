# FastPack specification

FastPack is an object serialization specification like JSON.  It is based heavily on the MessagePack specification but includes the following changes:
 * Little-endian instead of big-endian to match modern systems
 * Modified to allow parsers can to skip over nested maps and/or arrays while reading data within having to parse that data
 * Removal of extension types, as well short fixed width arrays and maps (since length in bytes is too small to be useful)
 * Addition common SQL data types including Decimal types and Date/Type types

## Table of contents

* FastPack specification
  * [Type system](#types)
      * [Limitation](#types-limitation)
  * [Formats](#formats)
      * [Overview](#formats-overview)
      * [Notation in diagrams](#formats-notation)
      * [nil format family](#formats-nil)
      * [bool format family](#formats-bool)
      * [int format family](#formats-int)
      * [float format family](#formats-float)
      * [str format family](#formats-str)
      * [bin format family](#formats-bin)
      * [array format family](#formats-array)
      * [map format family](#formats-map)
      * [decimal format family](#formats-decimal)
      * [dates format family](#formats-date)

<a name="types"/>
## Type system

* Types
  * **Integer** represents an integer
  * **Nil** represents nil
  * **Boolean** represents true or false
  * **Float** represents a floating point number
  * **Raw**
      * **String** extending Raw type represents a UTF-8 string
      * **Binary** extending Raw type represents a byte array
  * **Array** represents a sequence of objects
  * **Map** represents key-value pairs of objects
  * **Decimal** Exact decimal values
  * **Date/Time** Date, Time and interval values.

<a name="types-limitation"/>
### Limitation

* a value of an Integer object is limited from `-(2^63)` upto `(2^64)-1`
* a value of a Float object is IEEE 754 single or double precision floating-point number
* maximum length of a Binary object is `(2^32)-1`
* maximum byte size of a String object is `(2^32)-1`
* String objects may contain invalid byte sequence and the behavior of a deserializer depends on the actual implementation when it received invalid byte sequence
    * Deserializers should provide functionality to get the original byte array so that applications can decide how to handle the object



<a name="formats"/>
## Formats

<a name="formats-overview"/>
### Overview

<table>
  <tr><th>format name</th><th>first byte (in binary)</th><th>first byte (in hex)</th></th></tr>
  <tr><td>positive fixint</td><td>0xxxxxxx</td><td>0x00 - 0x7f</td></tr>
  <tr><td>(never used)</td><td>100xxxxx</td><td>0x80 - 0x9f</td></tr>
  <tr><td>fixstr</td><td>101xxxxx</td><td>0xa0 - 0xbf</td></tr>
  <tr><td>nil</td><td>11000000</td><td>0xc0</td></tr>
  <tr><td>(never used)</td><td>11000001</td><td>0xc1</td></tr>
  <tr><td>false</td><td>11000010</td><td>0xc2</td></tr>
  <tr><td>true</td><td>11000011</td><td>0xc3</td></tr>
  <tr><td>bin 8</td><td>11000100</td><td>0xc4</td></tr>
  <tr><td>bin 16</td><td>11000101</td><td>0xc5</td></tr>
  <tr><td>bin 32</td><td>11000110</td><td>0xc6</td></tr>
  <tr><td>date</td><td>11000111</td><td>0xc7</td></tr>
  <tr><td>time</td><td>11001000</td><td>0xc8</td></tr>
  <tr><td>interval</td><td>11001001</td><td>0xc9</td></tr>
  <tr><td>float 32</td><td>11001010</td><td>0xca</td></tr>
  <tr><td>float 64</td><td>11001011</td><td>0xcb</td></tr>
  <tr><td>uint 8</td><td>11001100</td><td>0xcc</td></tr>
  <tr><td>uint 16</td><td>11001101</td><td>0xcd</td></tr>
  <tr><td>uint 32</td><td>11001110</td><td>0xce</td></tr>
  <tr><td>uint 64</td><td>11001111</td><td>0xcf</td></tr>
  <tr><td>int 8</td><td>11010000</td><td>0xd0</td></tr>
  <tr><td>int 16</td><td>11010001</td><td>0xd1</td></tr>
  <tr><td>int 32</td><td>11010010</td><td>0xd2</td></tr>
  <tr><td>int 64</td><td>11010011</td><td>0xd3</td></tr>
  <tr><td>decimal9</td><td>11010100</td><td>0xd4</td></tr>
  <tr><td>decimal18</td><td>11010101</td><td>0xd5</td></tr>
  <tr><td>decimal28</td><td>11010110</td><td>0xd6</td></tr>
  <tr><td>decimal38</td><td>11010111</td><td>0xd7</td></tr>
  <tr><td>timestamp</td><td>11011000</td><td>0xd8</td></tr>
  <tr><td>str 8</td><td>11011001</td><td>0xd9</td></tr>
  <tr><td>str 16</td><td>11011010</td><td>0xda</td></tr>
  <tr><td>str 32</td><td>11011011</td><td>0xdb</td></tr>
  <tr><td>array 16</td><td>11011100</td><td>0xdc</td></tr>
  <tr><td>array 32</td><td>11011101</td><td>0xdd</td></tr>
  <tr><td>map 16</td><td>11011110</td><td>0xde</td></tr>
  <tr><td>map 32</td><td>11011111</td><td>0xdf</td></tr>
  <tr><td>negative fixint</td><td>111xxxxx</td><td>0xe0 - 0xff</td></tr>
</table>


<a name="formats-notation"/>
### Notation in diagrams

    one byte:
    +--------+
    |        |
    +--------+
    
    a variable number of bytes:
    +========+
    |        |
    +========+
    
    variable number of objects stored in FastPack format:
    +~~~~~~~~~~~~~~~~~+
    |                 |
    +~~~~~~~~~~~~~~~~~+
    
`X`, `Y`, `Z` and `A` are the symbols that will be replaced by an actual bit.

<a name="formats-nil"/>
### nil format

Nil format stores nil in 1 byte.

    nil:
    +--------+
    |  0xc0  |
    +--------+

<a name="formats-bool"/>
### bool format family

Bool format family stores false or true in 1 byte.

    false:
    +--------+
    |  0xc2  |
    +--------+
    
    true:
    +--------+
    |  0xc3  |
    +--------+

<a name="formats-int"/>
### int format family

Int format family stores an integer in 1, 2, 3, 5, or 9 bytes.

    positive fixnum stores 7-bit positive integer
    +--------+
    |0XXXXXXX|
    +--------+
    
    negative fixnum stores 5-bit negative integer
    +--------+
    |111YYYYY|
    +--------+
    
    * 0XXXXXXX is 8-bit unsigned integer
    * 111YYYYY is 8-bit signed integer

    uint 8 stores a 8-bit unsigned integer
    +--------+--------+
    |  0xcc  |ZZZZZZZZ|
    +--------+--------+
    
    uint 16 stores a 16-bit little-endian unsigned integer
    +--------+--------+--------+
    |  0xcd  |ZZZZZZZZ|ZZZZZZZZ|
    +--------+--------+--------+
    
    uint 32 stores a 32-bit little-endian unsigned integer
    +--------+--------+--------+--------+--------+
    |  0xce  |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|
    +--------+--------+--------+--------+--------+
    
    uint 64 stores a 64-bit little-endian unsigned integer
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0xcf  |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+

    int 8 stores a 8-bit signed integer
    +--------+--------+
    |  0xd0  |ZZZZZZZZ|
    +--------+--------+
    
    int 16 stores a 16-bit little-endian signed integer
    +--------+--------+--------+
    |  0xd1  |ZZZZZZZZ|ZZZZZZZZ|
    +--------+--------+--------+
    
    int 32 stores a 32-bit little-endian signed integer
    +--------+--------+--------+--------+--------+
    |  0xd2  |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|
    +--------+--------+--------+--------+--------+
    
    int 64 stores a 64-bit little-endian signed integer
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0xd3  |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+

<a name="formats-float"/>
### float format family

Float format family stores a floating point number in 5 bytes or 9 bytes.

    float 32 stores a floating point number in IEEE 754 single precision floating point number format:
    +--------+--------+--------+--------+--------+
    |  0xca  |XXXXXXXX|XXXXXXXX|XXXXXXXX|XXXXXXXX
    +--------+--------+--------+--------+--------+
    
    float 64 stores a floating point number in IEEE 754 double precision floating point number format:
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0xcb  |YYYYYYYY|YYYYYYYY|YYYYYYYY|YYYYYYYY|YYYYYYYY|YYYYYYYY|YYYYYYYY|YYYYYYYY|
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+
    
    where
    * XXXXXXXX_XXXXXXXX_XXXXXXXX_XXXXXXXX is a little-endian IEEE 754 single precision floating point number
    * YYYYYYYY_YYYYYYYY_YYYYYYYY_YYYYYYYY_YYYYYYYY_YYYYYYYY_YYYYYYYY_YYYYYYYY is a little-endian
      IEEE 754 double precision floating point number


<a name="formats-str"/>
### str format family

Str format family stores an byte array in 1, 2, 3, or 5 bytes of extra bytes in addition to the size of the byte array.

    fixstr stores a byte array whose length is upto 31 bytes:
    +--------+========+
    |101XXXXX|  data  |
    +--------+========+
    
    str 8 stores a byte array whose length is upto (2^8)-1 bytes:
    +--------+--------+========+
    |  0xd9  |YYYYYYYY|  data  |
    +--------+--------+========+
    
    str 16 stores a byte array whose length is upto (2^16)-1 bytes:
    +--------+--------+--------+========+
    |  0xda  |ZZZZZZZZ|ZZZZZZZZ|  data  |
    +--------+--------+--------+========+
    
    str 32 stores a byte array whose length is upto (2^32)-1 bytes:
    +--------+--------+--------+--------+--------+========+
    |  0xdb  |AAAAAAAA|AAAAAAAA|AAAAAAAA|AAAAAAAA|  data  |
    +--------+--------+--------+--------+--------+========+

    where
    * XXXXX is a 5-bit unsigned integer which represents N
    * YYYYYYYY is a 8-bit unsigned integer which represents N
    * ZZZZZZZZ_ZZZZZZZZ is a 16-bit little-endian unsigned integer which represents N
    * AAAAAAAA_AAAAAAAA_AAAAAAAA_AAAAAAAA is a 32-bit little-endian unsigned integer which represents N
    * N is the length of data

<a name="formats-bin"/>
### bin format family

Bin format family stores an byte array in 2, 3, or 5 bytes of extra bytes in addition to the size of the byte array.

    bin 8 stores a byte array whose length is upto (2^8)-1 bytes:
    +--------+--------+========+
    |  0xc4  |XXXXXXXX|  data  |
    +--------+--------+========+
    
    bin 16 stores a byte array whose length is upto (2^16)-1 bytes:
    +--------+--------+--------+========+
    |  0xc5  |YYYYYYYY|YYYYYYYY|  data  |
    +--------+--------+--------+========+
    
    bin 32 stores a byte array whose length is upto (2^32)-1 bytes:
    +--------+--------+--------+--------+--------+========+
    |  0xc6  |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|  data  |
    +--------+--------+--------+--------+--------+========+

    where
    * XXXXXXXX is a 8-bit unsigned integer which represents N
    * YYYYYYYY_YYYYYYYY is a 16-bit little-endian unsigned integer which represents N
    * ZZZZZZZZ_ZZZZZZZZ_ZZZZZZZZ_ZZZZZZZZ is a 32-bit little-endian unsigned integer which represents N
    * N is the length of data

<a name="formats-array"/>
### array format family

Array format family stores a sequence of elements in 3 or 5 bytes of extra bytes in addition to the elements.

    array 16 stores an array whose length is upto (2^16)-1 bytes (YYYY):
    +--------+--------+--------+~~~~~~~~~~~~~~~~~~~~~~~~~+
    |  0xdc  |YYYYYYYY|YYYYYYYY|    N bytes of objects   |
    +--------+--------+--------+~~~~~~~~~~~~~~~~~~~~~~~~~+
    
    array 32 stores an array whose length is upto (2^32)-1 elements:
    +--------+--------+--------+--------+--------+~~~~~~~~~~~~~~~~~~~~~~~~~+
    |  0xdd  |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|    N bytes of objects   |
    +--------+--------+--------+--------+--------+~~~~~~~~~~~~~~~~~~~~~~~~~+
    
    where
    * YYYYYYYY_YYYYYYYY is a 16-bit little-endian unsigned integer which represents N
    * ZZZZZZZZ_ZZZZZZZZ_ZZZZZZZZ_ZZZZZZZZ is a 32-bit little-endian unsigned integer which represents N
        N is the size of a array

<a name="formats-map"/>
### map format family

Map format family stores a sequence of key-value pairs in 3, or 5 bytes of extra bytes in addition to the key-value pairs.

    
    map 16 stores a map whose length is upto (2^16)-1 bytes
    +--------+--------+--------+~~~~~~~~~~~~~~~~~~~~~~~~+
    |  0xde  |YYYYYYYY|YYYYYYYY|   N bytes of objects   |
    +--------+--------+--------+~~~~~~~~~~~~~~~~~~~~~~~~+
    
    map 32 stores a map whose length is upto (2^32)-1 bytes
    +--------+--------+--------+--------+--------+~~~~~~~~~~~~~~~~~~~~~~~~+
    |  0xdf  |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|   N bytes of objects   |
    +--------+--------+--------+--------+--------+~~~~~~~~~~~~~~~~~~~~~~~~+
    
    where
    * XXXX is a 4-bit unsigned integer which represents N
    * YYYYYYYY_YYYYYYYY is a 16-bit little-endian unsigned integer which represents N
    * ZZZZZZZZ_ZZZZZZZZ_ZZZZZZZZ_ZZZZZZZZ is a 32-bit little-endian unsigned integer which represents N
    * N is the size of a map in bytes
    * odd elements in objects are keys of a map
    * the next element after a key is its associated value

<a name="formats-decimal"/>

### Decimal format family

    decimal9 stores an exact decimal up to 9 digits in size.
    +--------+--------+--------+--------+--------+--------+
    |  0xd4  |PPPPSSSS|ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ|
    +--------+--------+--------+--------+--------+--------+

    decimal18 stores an exact decimal up to 18 digits in size.
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0xd5  |PPPPPPPP|SSSSSSSS|ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ|
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+

    decimal28 stores an exact decimal up to 28 digits in size.
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0xd6  |PPPPPPPP|SSSSSSSS|ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ|
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+

    decimal38 stores an exact decimal up to 38 digits in size.
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0xd7  |PPPPPPPP|SSSSSSSS|ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ ZZZZZZZZ|
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    
    * PPPP... represents the bits focused on providing scale.
    * SSSS... represents the bits focused on providing precision.
    * ZZZZ... represents a two's complement number composed of the set number of bytes.
        

<a name="formats-date"/>
### Date and Time Formats

    timestamp Stored as milliseconds since Unix epoch (signed little-endian)
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0xd8  |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+    
  
    date Stored as days since Unix epoch (signed little-endian)
    +--------+--------+--------+--------+--------+
    |  0xc7  |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|
    +--------+--------+--------+--------+--------+

    time Stored as milliseconds since midnight (signed little-endian)
    +--------+--------+--------+--------+--------+
    |  0xc8  |ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|ZZZZZZZZ|
    +--------+--------+--------+--------+--------+  
    
    interval Duration of time stored as months, days and milliseconds (each stored as little endian)
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+
    |  0xc9  |NNNNNNNN NNNNNNNN NNNNNNNN NNNNNNNN DDDDDDDD DDDDDDDD DDDDDDDD DDDDDDDD MMMMMMMM MMMMMMMM MMMMMMMM MMMMMMMM|
    +--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+--------+




