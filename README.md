FastPack
===========

FastPack is an object serialization specification like JSON.  It is based heavily on the MessagePack specification but includes the following changes:
 * Little-endian instead of big-endian to match modern systems
 * Modified to allow parsers can to skip over nested maps and/or arrays while reading data without having to parse that data
 * Removal of extension types, as well short fixed width arrays and maps (since length in bytes is too small to be useful)
 * Addition common SQL data types including Decimal types and Date/Type types

