FastPack
===========

FastPack is an object serialization specification like JSON.  It is based heavily on the MessagePack specification.  However, it is updated so that more common SQL data types can be directly encoded.  It is also primarily little endian instead of big endian to match modern systems.  It is also modified to allow parsers can to skip over nested maps and/or arrays while reading data within having to parse that data.  


