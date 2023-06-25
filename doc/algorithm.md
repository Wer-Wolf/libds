# Doublespace decompression algorithm

## Basic operation

The doublespace decompression algorithm is based on the well-known LZ77 algorithm,
meaing decompression works on tuples containing a length and an offset, or an literal.

If the tuple is an literal, its value is placed directly inside the output buffer. Otherwise,
the data already inside the output buffer is copied according to the length and offset values.
The length specifies how many bytes should be copied, while the offset specifies the starting
point for the data to be copied from (start = current position - offset).
Should the copy operation exceed the current position inside the output buffer, then it will
wrap around.

There are a couple of special offset values:
* `0x0`: Invalid offset, should be ignored
* `0x113f`: Special offset used for synchronization, usually occurs every 512 bytes and at the
            end of output data

## Bit stream format

### Header

The header consists of the following information:
* a magic number containing the two letters `DS` (`0x5344` little endian)
* a 16 bit big endian number containg the version of the algorithm (1 - 3 are known to work)

### Tuple stream

The first two bits of each tuple describe the content of the following data:
* `0x0`: Standard length/offet tuple
* `0x1`: Big literal
* `0x2`: Small literal
* `0x3`: Extended length/offset tuple

**All length/offset values are in little endian!**

#### Literals

Each literal contains 7 bits of data. A small literal has the 8th bit set to 0, while a big
literal has the 8th bit set to 1.

#### Standard length/offset tuple

Such a tuple contain a 6 bit offset followed by the length value.

#### Extended length/offset tuple

Such a tuple starts with a special bit. If this bit is true, then it is followed by a 12 bit offset,
otherwise it is followed by a 8 bit offset. Both offsets need an additional offset applied onto them
(320 for the 12 bit offset and 64 for the 8 bit offset). The offset value is then followed by a length
value.

#### Length value

The length value starts with a number of zero bits (between 0 and 8), terminated by a one bit, which sginal the length of the following length value (in bits). For each length (0, 1, 2, 3, 4, 5, 6, 7,8), an offset needs to be applied (2, 3, 5, 9, 17, 33, 65, 129, 257).