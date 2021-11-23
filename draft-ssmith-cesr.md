---

title: "Composable Event Streaming Representation (CESR)"
abbrev: "CESR"
docname: draft-ssmith-cesr-latest
category: info

ipr: trust200902
area: TODO
workgroup: TODO Working Group
keyword: Internet-Draft

stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    name: S. Smith
    organization: ProSapien LLC
    email: sam@prosapien.com

normative:

informative:
  KERI:
    target: https://arxiv.org/abs/1907.02143
    title: Key Event Receipt Infrastructure (KERI)
    author:
        ins: S. Smith
        name: Samuel M. Smith
        org: ProSapien LLC
    date: 2021


tags: IETF, CESR, SAID, KERI, ACDC

--- abstract

TODO Abstract


--- middle

# Introduction

One way to better secure Internet communications is to use cryptographically verifiable primitives and data structures in communication. These provide essential building blocks for zero trust computing and networking architectures. Traditionally cryptographic primitives that include but are not limited to digests, salts, seeds (private keys), public keys, and digital signatures have been largely represented in some type of binary encoding. This limits their usability in domains or protocols that are human centric or equivalently that only support text-printable characters. These domains include source code, JSON, system logs, audit logs, Ricardian contracts, and human readable text documents of many types. 

Generic binary-to-text or simply textual encodings such as Base64 do not provide any information about the type or size of the underlying cryptographic primitive. Base64 only provides value information. More recently Base58Check was developed as a fit for purpose textual encoding of cryptographic primitives  for shared distributed ledger applications that in addition to value may include information about the type and in some cases the size of the underlying cryptographic primitive. But each application may use a non-interoperable encoding of type and optionally size. Interestingly because a binary encoding may include as a subset some codes that are in the text-printable compatible subset of ASCII (Latin-1,UTF-8)one may serendipoudously find, for a given cryptographic primitive, a text-printable type code from a binary code table like Multihash for example. Indeed some Base58Check applications take advantage of binary Multihash but serindipidous text compatible type codes. Serindipoudous text encodings in binary code tables, however do not work in general for size. So the approach is not universally applicable and is no substitute for a true textual encoding protocol fro cryptographic primitives.

In general there is no standard text based encoding protocol that provides universal type, size, and value encoding for cryptographic primitives. Providing this capability is the primary motivation for the encoding protocol defined herein.

Importantly, a textual encoding that includes type, size, and value is self-framing. A self-framing text primitive may be parsed without needing any additional delimiting characters. Thus a stream of concatenated primitives may be individually parsed without the need to encapsulate the primitives inside textual delimiters or envelopes. Thus a textual self-framing encoding provides the core capability for a streaming text protocol. Although textual encoding of cryptographic primitives is the primary motivation for the protocol defined herein, this protocol is sufficiently flexible and extensible to support other useful data types, such as, integers of various sizes, floating point numbers, date-times as well as generic text. Thus this protocol is generally useful to encode in text data data structures of all types not merely those that contain cryptographic primitives.

Textual encodings have numerous usability advantages over binary encodings. The one advantage, however, a binary encoding has over text is compactness. An encoding protocol that has the property we call *text-binary concatentation composability* or more succinctly *composability*, enables both the usability of text and the compactness of binary. Composability may be the most uniquely innovative and useful feature of the encoding protocol defined herein.

## Composability

Composability as defined here  is short for text-binary concatenation composability. An encoding has text-binary concatenation composability when any set of self-framing concatenated primitives expressed in either the text domain or binary domain may be converted as a group to the other domain and back again without loss. Essentially composability provides round-trippable lossless conversion between text and binary representations of any set of concatenated primitives. The property enables a stream processor to safely convert en-masse a a stream of text primitives to binary for compact transmission that the stream processor at the other end may safely convert back to text for further processing or archival storage as text. With the addition of group framing codes as composable primitives, such a composable encoding protocol enables pipelining (multi-plexing and de-multiplexing) of streams in either text or compact binary. This allows management at scale for high-bandwidth applications that benefit from core affinity off-loading of streams.  

## Abstract Domain Representations

The cryptographic primitives defined here (i.e. CESR) inhabit three different domains each with a different representation.  The first domain we call streamable text or *text* and is denoted as *T*. The third domain we call streamable binary or *binary* and is denoted as *B*. Composability is defined between the *T* and *B* domains. The third domain we call *raw* and is denoted as *R*. The third domain is special because primitives in this domain are represented by a pair or two tuple of values namely *(text code, raw binary)*. The *text code* element of the *R* domain pair is string of one or more text characters that provides the type and size information for the encoded primitive when in the *T* domain. Actual use of cryptographic primitives happens in the *R* domain using the *raw binary* element of the `(code, raw binary)` pair. Cryptographic primitive values are usually represented as strings of bytes that represent very large integers. Cryptographic libraries typically assume that the inputs and outputs of their functions will be such strings of bytes. The *raw binary* element of the *R* domain pair is such a string of bytes. 

A given primitive in the *T* domain is denoted with `t`.  A member of an indexed set of primitives in the *T* domain is denoted with `t[k]`. Likewise a given primitive in the *B* domain is denoted with `b`. A member of an indexed set of primitives in the *B* domain is denoted with `b[k]`. Similarly a given primitive in the *R* domain is denoted with `r`. A member of indexed set of primitives in the *R* domain is denoted with  `r[k]`.

### Transformations Between Domains

Although, the composability property, mentioned above but described in detail below, only applies to conversions back and forth between the *T*, and *B*, domains, conversions between the *R*, and *T* domains as well as conversions between the *R* and *B* domains are also defined and supported by the protocol. As a result there is a total of six transformations, one in each direction between the three domains.

Let `T(B)` denote the abstract transformation function from the *B* domain to the *T* domain. This is the dual of `B(T)` below.

Let `B(T)` denote the abstract transformation function from the *T* domain to the *B* domain. This is the dual of `T(B)` above.

Let `T(R)` denote the abstract transformation function from the *R* domain to the *T* domain. This is the dual of `R(T)` below.

Let `R(T)` denote the abstract transformation function from the *T* domain to the *R* domain. This is the dual of `T(R)` above.

Let `B(R)` denote the abstract transformation function from the *R* domain to the *B* domain. This is the dual of `R(B)` below.

Let `R(B)` denote the abstract transformation function from the *B* domain to the *R* domain. This is the dual of `B(R)` above.

Given these transformations we can complete of circuit of transformations that starts in any of the three domains and then crosses over the other two domains in either direction. For example starting in the *R* domain we can traverse a circuit that crosses into the *T* and *B* domains and then crosses back into the *R* domain as follows:

~~~
R->T(R)->T->B(T)->B->R(B)->R
~~~

Likewise starting in the *R* domain we can traverse a circuit that crosses into the *B* and *T* domains and then crosses back into the *R* domain as follows:

~~~
R->B(R)->B->T(B)->T->R(T)->R
~~~

### Concatenation Composability Property

Let `+` represent concatenation. Concatenation is associative and may be applied to any two primitives or any two groups or sets of concatenated primitives. For example: 
~~~
t[0] + t[1] + t[2] + t[3] = (t[0] + t[1]) + (t[2] + t[3])
~~~

If we let `cat(x[k])` denote the concatenation of all elements of a set of indexed primitives `x[k]` where each element is indexed by a unique value of `k` then we can denote the transformation between domains of a concatenated set of primitives as follows:

Let `T(cat(b[k]))` denote the concrete transformation of a given concatenated set of primitives, `cat(b[k])` from the *B* domain to the *T* domain.

Let `B(cat(b=t[k]))` denote the concrete transformation of a given concatenated set of primitives, `cat(t[k])` from the *T* domain to the *B* domain.

The concatentation composability property between *T* and *B* is expressed as follows:

Given a set of primitives `b[k]` and `t[k]` and transformations `T(B)` and `B(T)` such that `t[k] = T(b[k])` and `b[k] = B(t[k])` for all `k`, then `T(B)` and `B(T)` are jointly concatenation composable if and only if, 
~~~
T(cat(b[k]))=cat(T(b[k])) and B(cat(t[k]))=cat(B(t[k])) for all k
~~~

Basically composability (over concatenation) means that the transformation of a set (as a whole) of concatenated primitives is equal to the concatentation of the set of individually transformed primitives. 

For example, suppose we have two primitives in the text domain, namely, `t[0]` and `t[1]` that each respectively transform to primitives in the binary domain, namely, `b[0]` and `b[1]`. The transformation duals `B(T)` and `T(B)` are composable if and only if,
~~~
B(t[0] + t[1]) = B(t[0]) + B(t[1]) = b[0] + b[1]
~~~
and
~~~
T(b[0] + b[1]) = T(b[0]) + T(b[1]) = t[0] + t[1]
~~~

The composability property allows us to create arbitrary compositions of primitives via concatenation in either the *T* or *B* domain and then convert the composition en masse to the other domain and then de-concatenate the result without loss. The self-framing property of the primitives enables de-concatenation. 

The composability property is an essential building block for streaming in either domain. The use of framing primitives that count other primitives enables multiplexing and demultiplexing of arbitrary groups of primitives for pipelining and/or on or off loading of streams. The text domain provides usability and the binary domain provides compactness. Composability allows efficient conversion of composed (concatenated) groups of primitives without having to individually parse each primitive.

## Concrete Domain Representations

Text, *T*, domain representations in CESR use only the characters from the the URL and filename safe variant of the IETF RFC-4648 Base64 standard. Unless otherwise indicated all references to Base64 (RFC-4648) in this document imply the URL and filename safe variant. The URL and filename safe variant of Base64 uses in order the 64 characters `A` through `Z`, `a` through `z`, `-`, and `_` to encode 6 bits of information. In addition Base64 uses the `=` character for padding but CESR does not use the `=` character for any purpose.

Base64 by itself does not satisfy the composability property. 
In CESR, both *T* and *B* domain representations include a prepended framing code prefix that is structured in such a way as to ensure composability. 

Suppose for example we wanted to use naive Base64 characters in the text domain and naive binary bytes in the binary domain. For the sake of the example we will call these naive text and naive binary encodings and domains. Recall that a byte encodes 8 bits of information and a Base64 character encodes 6 bits information. Furthermore suppose that we have three primitives denoted `x`, `y`, and `z` in the naive binary domain with lengths of 1, 2, and 3 bytes respectively.

In the following diagrams we denote each byte in a naive binary primitive with zero based most significant bit first indicies.  For example, `b1` is bit one `b0` is bit zero and `B0` for byte zero, `B1` for byte 1, etc.

The byte and bit level diagram for `x` is shown below where we use `X` to denote its bytes:

~~~
|           X0          |
|b7:b6:b5:b4:b3:b2:b1:b0|
~~~

Likewise for `y` below:
~~~
|           Y0          |           Y1          |
|b7:b6:b5:b4:b3:b2:b1:b0|b7:b6:b5:b4:b3:b2:b1:b0|
~~~

And finally for `z` below:
~~~
|           Z0          |           Z1          |           Z2          |
|b7:b6:b5:b4:b3:b2:b1:b0|b7:b6:b5:b4:b3:b2:b1:b0|b7:b6:b5:b4:b3:b2:b1:b0|
~~~

When doing a naive Base64 conversion of a naive binary primitive, one Base64 character represents only six bits from a given byte. In the following diagrams each character of a Base64 conversion is denoted with zero based most significant character first indicies. 

Therefore to encode `x` in Base64, for example, requires at least two Base64 characters because the zeroth character only captures the six bits from the first byte and another character is needed to capture the other two bits. The convention in Base64 is use a Base64 character where the non-coding bits are zeros. This is diagrammed as follows:

~~~
|           X0          |
|b7:b6:b5:b4:b3:b2:b1:b0|
|        C0       |        C1       |
~~~

Naive Base64 encoding always pads each individual conversion of a string of bytes to an even multiple of four characters. This provides something is not true composability but does ensure that multiple distinct concatentated conversions are separable. It may be described as a sort of one-way composability. So with pad characters, denoted by replacing the spaces with `=` characters, the Base64 conversion of `x` is as follows:

~~~
|           X0          |
|b7:b6:b5:b4:b3:b2:b1:b0|
|        C0       |        C1       |========C2=======|========C3=======|

~~~

Likewise `y` requires at least 3 Base64 characters to capture all of its 16 bits as follows:

~~~
|           Y0          |           Y1          |
|b7:b6:b5:b4:b3:b2:b1:b0|b7:b6:b5:b4:b3:b2:b1:b0|
|        C0       |        C1       |        C1       |
~~~

Alignment on a 4 character boundary requires one pad character this becomes:

~~~
|           Y0          |           Y1          |
|b7:b6:b5:b4:b3:b2:b1:b0|b7:b6:b5:b4:b3:b2:b1:b0|
|        C0       |        C1       |        C2       |========C3=======|
~~~

Finally because `z`  requires exactly four Base64 characters to capture all of its 24 bits, there are no pad characters needed.

~~~
|           Z0          |           Z1          |           Z2          |
|b7:b6:b5:b4:b3:b2:b1:b0|b7:b6:b5:b4:b3:b2:b1:b0|b7:b6:b5:b4:b3:b2:b1:b0|
|        C0       |        C1       |        C2       |        C3       |
~~~

Suppose we concatenate `x + y` into a three byte composition in the naive binary domain before Base64 encoding the concatentated whole. We have the following:

~~~
|           X0          |           Y0          |           Y1          |
|b7:b6:b5:b4:b3:b2:b1:b0|b7:b6:b5:b4:b3:b2:b1:b0|b7:b6:b5:b4:b3:b2:b1:b0|
|        C0       |        C1       |        C2       |        C3       |
~~~

We see that the least significant two bits of X0 are encoded into the same character, `C1` as the most significant four bits of `Y0`. Therefore, a text domain parser is unable to cleanly de-concatenate the conversion of `x + y` into separate text domain primitives. Thus naive binary to Base64 conversion does not satisfy the composability constraint.

Suppose instead we start in the text domain with primitives `u` and `v` of lengths 1 and 3 characters respectively.
If we concatenate these two primitives as `u + v` in text domain and then convert as a whole to naive binary. We have the following:

~~~
|        U0       |        V0       |        V1       |        V2       |
|b7:b6:b5:b4:b3:b2:b1:b0|b7:b6:b5:b4:b3:b2:b1:b0|b7:b6:b5:b4:b3:b2:b1:b0|
|           B0          |           B1          |           B2          |
~~~

We see that all six bits of information in `U0` is included with the least significant two bits of information in `V0` in `B0`. Therefore a binary domain parser is unable to cleanly de-concatenate the conversion of `u + v` into separate binary domain primitives. Thus naive Base64 to binary conversion does not satisfy the composability constraint. 

Indeed the composability property is only satisfied if each primitive in the *T* domain is an integer multiple of four Base64 characters and each primitive in the *B* domain is an integer multiple of three bytes. Each of Four Base64 characters and three bytes capture twenty-four bits of information. Twenty-four is the least common multiple of six and eight. Therefore primitive lengths that integer multiples of either four Base64 characters or three bytes in their respective domains capture integer multiples of twenty-four bits of information. Given that constraint, conversion of concatenated primitives in one domain never result in two adjacent primitives sharing the same byte or character in the converted domain. 

To elaborate, when converting streams made up of concatenated primitives back and forth between the *T* and *B* domains, the converted results will not align on byte or character boundaries at the end of each primitive unless the primitives themselves are integer multiples of twenty-four bits of information. In other words all primitives must be aligned on twenty-four bit boundaries to satisfy the composibility property. This means that the minimum length of any primitive in the B domain is three bytes and the minimum length of any primitive in the T domain is four Base64 characters.


### Stable Text Type Codes

There are many coding schemes that could satisfy the composability constraint of alignment on 24 bit boundaries. The main reason for using a *T* domain centric encoding is higher usability or human friendliness. Indeed a primary design goal of CESR is to select an encoding approach that provides such high usability or human friendliness in the *T* domain. This type of usability goal is simpley not realizable in the *B* domain. The B domain's purpose is merely to provide convenient compactness at scale. We believe usability in the *T* domain is maximized when the type portion of the prepended framing code is *stable* or *invariant*. Stable type coding makes it much easier to recognize primitives of a given type  when debugging source, reading messages, or documents in the *T* domain that include encoded primitives. This is true even when those primitives have different lengths or values. For primitive types that have fixed lengths, i.e. all primitives of that type have the same length, stable type coding aids not only visual type but visual size recognition.

Usability of stable type coding is maximized when the type portion appears first in the framing code. Stability also requires that for a given type, the type coding portion must consume a fixed integer number of characters in the *T* domain. To clarify, as used here, stable type coding in the *T* domain never shares information bits with either length or value coding in any given framing code character and appears first in the framing code. Stable type coding in the *T* domain translates to stable type coding in the *B* domain except that the type coding portion of the framing code may not respect byte boundaries. This is an acceptable tradeoff because binary domain parsing tools easily accommodate bit fields and bit shifts while text domain parsing tools no not. By in large text domain parsing tools only process whole characters. This is another reason to impose a stability constraint on the *T* domain type coding instead of the *B* domain.

### Multiple Code Table Approach

Our design goals for framing codes include minimizing the framing code size for the most frequently used (most popular) codes while also supporting a sufficiently comprehensive set of codes for all foreseeable current and future applications. This requires a high degree of both flexibility and extensibility. We believe this is best achieved with multiple code tables each with a different coding scheme that is optimized for a different set of features instead of a single one-size-fits-all scheme. A specification that supports multiple coding schemes may appear on the surface to be much more complex to implement but careful design of the coding schemes can reduce implementation complexity by using a relatively simple single integrated parse and conversion table. Parsing in any given domain given stable type codes may then be implemented with a single function that simply reads the appropriate type selector in the table to know how to parse and convert the rest of primitive.

## Text Coding Scheme Design

### Text Code Size
Recall from above, the R domain representation is a pair`(text code, raw binary)`. The text code is stable and begins with one or more Base64 characters that provide the primitive type may also include one or more additional characters that provide the length. The actual usable cryptographic material is provided by the *raw binary* element. The corresponding *T* domain representation of this pair is created by first converting the *raw binary* element to Base64, then stripping off any Base64 pad characters then finally prepending the text code element to the result of the conversion. 

When the length of a given naive binary string is not an integer multiple of three bytes, standard Base64 conversion software appends one or two pad characters to the resultant Base64 conversion. 

With pad characters, a set of Base64 strings resulting from the subsequent conversions of a set of binary strings could be concatenated and then converted back to binary en masse while preserving byte boundaries. In order to preserve byte boundaries the pad characters MUST be stripped in the conversion.  Because the pad characters are stripped, this approach does not provide two-way or true composability as defined above. The number of pad characters is a function of the length of the binary string. Let *N* be the length the string. When `N mod 3 = 1` then there are 8 bits in remainder that must be encoded into Base64. Recall the examples above, a single byte (8 bits) require two Base64 characters. The first encodes  6 bits and the second the remaining 2 bits for a total of 8 bits. The last character is selected such that its non-coding 4 bits are zero. Thus two additional pad characters are required to pad out the resulting conversion so that its length is an integer multiple of 4 Base64 characters. When `N mod 3 = 2` then two bytes (16 bits) require three Base64 characters. The first two encode 6 bits each (for 12 bits) and the third encodes the remaining 4 bits for a total of 16. The last character is selected such that its non-coding 2 bits are zero. Thus one additional pad character is required to pad out the resulting conversion so that is length is an integer multiple of 4 characters. When `N mod 3 = 0` then the binary string is aligned on a 24 bit boundary and no pad characters are required to ensure the length of the Base64 conversion is an integer multiple of 4 characters.

The number of requred Base64 pad characters to ensure that a given conversion to Base64 has a length that is an integer multiple of 4 may be computed with the following formula:

`ps = (3 - (N mod 3)) mod 3)`, where `ps` is the pad size and `N` is the size in bytes of the binary string.

Recall that composability is provided here by prepending text codes that are of the appropirate length to ensure 24 bit boundaries in both the *T* and the corresponding *B* domain. The advantage of this approach is that naive Base64 software tooling may be used to convert back and forth between the *T* and *B* domains, i.e. `T(B)` is naive Base64 encode and `B(T)` is naive Base64 decode. In other words CESR primitives are compatible with existing Base64 (RFC-4648) tooling. Whereas new software tooling is needed for conversions between the *R* and *T* domains, e.g. `T(R)` and `R(T` and the *R* and *B* domains, e.g. `B(R)` and `R(B)`.


This pad size computation is also useful for computing the size of the text codes. Because true composability also requires that the *T* domain value MUST be an integer multiple of 4 characters in length the size of the text code MUST also be function of the pad size, `ps`, and hence the length of the raw binary element, `N`. Thus the size of the text code in Base64 characters is a function of the equivalent pad size determined by the length `N mod 3` of the raw binary value. If we let *M* be a non-negative integer valued variable then we have three cases: 

|   Pad Size   |  Code Size  | 
|:------------:|------------:|
|0|4•M|
|1|4•M + 1|
|2|4•M + 2|

The minimum code sizes are 1, 2, and 4 characters for pad sizes of 1, 2, and 0 characters with *M* equal 0, 0, and 1 respectively. By increasing *M* we can have larger code sizes for a given pad size.


### Count, Group, or Frame Codes
As mentioned above one of the primary advantages of a composable encoding is that special framing codes can be specified to support groups of primitivies. Grouping enables pipelining. Other suitable terms for these special framing codes are *group codes* or *count codes*. These are suitable because they can be used to count characters, primitives in a group, or count groups of primitives in a larger group. We can also use count codes as separators to organize a stream of primitives or to interleave non-native serializations. A count code is its own primitive. But it is a primitive that does not include a raw binary value, only the text code. Because a count code's raw binary element is empty, its pad size is always 0. Thus a count code's size is always an integer multiple of 4 characters i.e. 4, 8, etc. 

### Interleaved Non-CESR Serializations
One extremely useful property of CESR is that special count codes enable CESR to be interleaved with other serializations. For example, Many applications use JSON, CBOR, or MsgPack (MGPK) to serialize flexible self-describing data structures based on hash maps, also know as dictionaries. With respect to hash map serializations, CESR primitives may appear in two different contexts. The first context is as a delimited text primitive inside of a hash map serialization. The delimited text may be either the key or value of a (key, value) pair. The second context is as a standalone serialization that is interleaved with hash map serializations in a stream. Special CESR count codes enable support for the second context of interleaving standalone CESR with other serializations.

### Cold Start Stream Parsing Problem

After a cold start a stream processor looks for framing information to know how to parse groups of elements in the stream. If that framing information is ambiguous then the parser may become confused and require yet another cold start. While processing a given stream a parser may become confused especially if a portion of the stream is malformed in some way. This usually requires flushing the stream and forcing a cold start to resynchronize the parser to subsequent stream elements. Better yet is a re-synchronization mechanism that does not require flushing the in-transit buffers but merely skipping to the next well defined stream element boundary in order to execute  cold start. Good cold start  re-synchronization is essential to robust performant stream processing.

For example, in TCP a cold start usually means closing and then reopening the TCP connection. This flushes the TCP buffers and sends a signal to the other end of the stream that may be interpreted as a restart or cold start. In UDP each packet is individually framed but a stream may be segmented into multiple packets so a cold start may require an explicit ack or nack to force a restart. 

Special CESR count codes support re-synchronization at each boundary between interleaved CESR and other serializations like JSON, CBOR, or MGPK 


#### Performant Resynchronization with Unique Start Bits

Given the popularity of three specific serializations, namely, JSON, CBOR, and MGPK, more fine grained serialization boundary detection for interleaving CESR may be highly beneficial both from a performation and robustness perspective. One way to provide this is by selecting the count code start bits such that there is always a unique (mutually distinct) set of start bits at each interleaved boundary between CESR, JSON, CBOR, and MGPK. 

Furthermore, it may also be highly beneficial to support in-stride switching between interleaved CESR text domain streams and CESR binary domain streams. In other words the start bits for count (framing) codes in both the *T* domain (Base64) and the *B* domain should be unique. This would provide the analogous equivalent of a UTF Byte Order Mark (BOM). A BOM enables a parser of UTF encoded documents to determine if the UTF codes are big endian or little endian [[19]]. In the CESR case this feature would enable a stream parser to know if a count code along with its associated counted or framed group of primitives are expressed in the *T* or *B* domain. Toghether these impose the constraint that the boundary start bits for interleaved text CESR, binary CESR, JSON, CBOR, and MGPK be mutually distinct.


Amongst the codes for map objects in the JSON, CBOR, and MGPK only the first three bits are fixed and not dependent on mapping size. In JSON a serialized mapping object always starts with `{`. This is encoded as `0x7b`. the first three bits are `0b011`. In CBOR the first three bits of the major type of the serialized mapping object are `0b101`. In MGPK (MsgPack) there are three different mapping object codes. The *FixMap* code starts with `0b100`. Both the *Map16* code and *Map32* code start with `0b110`.

So we have the set of four used starting tritets (3 bits) in numeric order of `0b011`, `0b100`, `0b101`, and `0b110`. This leaves four unused tritets, namely, `0b000`, `0b001`, `0b010`, and `0b111` that may be selected as the CESR count (framing) code start bits. In Base64 there are two codes that satisfy our constraints. The first is the dash character, `-`, encoded as `0x2d`. Its first three bits are `0b001`. The second is the underscore character,`_`, encoded as `0x5f`. Its first three bits are `0b010`. Both of these are distinct from the starting tritets of any of the JSON, CBOR, and MGPK encodings above. Moreover the starting tritet of the corresponding binary encodings of `-` and `_` is `0b111` which is also distinct from the all the others. To elaborate, Base64 uses `_` in position 62 or `0x3E` (hex) and uses `_` in position 63 or `0x3F` (hex) both of which have starting tritet of `0b111` 

This gives us two different Base64 characters, `-` and `_` we can use for the first character of any framing (count) code in the *T* domain. This also means we can have two different classes of framing (count) codes. This also provides a BOM like capability to determine if a framing code is expressed in the *T* or *B* domain. To clarify, if a stream  starts with the tritet `0b111` then the stream is *B* domain CESR and a stream parser would thereby know how to convert the first sextet of the stream to determine which of the two framing codes is being used, `0x3E` or `ox3F` . If on the other hand the framing code starts with either of the tritets `0b001` or `0b010` then the framing code is expressed in the *T* domain and a stream parser likewise would thereby know how to convert the first character (octet) of the framing code to determine which framing code is being used. Otherwise if a stream starts with  `0b100` then is JSON, with `0b101` then its CBOR and with either `0b011`,and `0b110` then its MGPK.

This is summaraized in the following table:


|   Starting Tritet   |  Serialization  | Character |
|:------------:|:------------:|:------------:|
|0b000|||
|0b001|CESR *T* Domain|`-`|
|0b010|CESR *T* Domain|`_`|
|0b011|JSON|`{`|
|0b100|MGPK||
|0b101|CBOR||
|0b110|MGPK||
|0b111|CESR *B* Domain||


JSON Mapping Object Delimeters  

https://www.json.org/json-en.html  

CBOR Mapping Object Codes  

https://en.wikipedia.org/wiki/CBOR  

MsgPack Mapping Object Codes  

https://github.com/msgpack/msgpack/blob/master/spec.md  

UTF Byte Order Mark (BOM)  

https://en.wikipedia.org/wiki/Byte_order_mark  



#### Stream Parsing Rules

Given this set of tritets (3 bits) we can express a requirement for well formed stream start and restart.

Each stream MUST start (restart) with one of five tritets:

1) A framing code in CESR *T* domain
2) A framing code in CESR *B* Domain.
3) A JSON encoded mapping.
4) A CBOR encoded Mapping.
5) A MGPK encoded mapping. 


A parser merely needs to examine the first tritet (3 bits) of the first byte of the stream start to determine which one of the five it is. When the first tritet is a framing code then, the remainder of framing code itself will include the remaining information needed to parse the attached group. When the first tritet indicates its JSON, CBOR, or MGPK, then the mapping's first field must be a version string that provides the remaining information needed to fully parse the associated encoded serialization. 

The stream MUST resume with a starting byte that starts with one of the 5 tritets, either another framing code expressed in the *T* or *B* domain or a new JSON, CBOR, or MGPK encoded mapping.

This provides an extremely compact and elegant stream parsing formula that generalizes not only supports CESR composabilty but also supports interleaving CESR with the most popular hash map serializations. 


### Compact Fixed Size Codes

Modern crypto fixed raw binary lengths. Fixed minimum crytographic streangth fixes minimum length more is just wasting size and computation. So only need one size for each operation (digest key derivation signature) for a given crypto suit
such as Ed25519. Most popular
Just so happens that 32 byte is the sweet spot for popularity.


Twelve code tables enough for the foreseeable future.

Any binary item whose length mod 3 is 1 will need 2 pad characters to makes its converted length a multiple of four characters. Any binary item whose length mod 3 is 2 will need 1 pad characters to makes its converted length a multiple of four characters. Consequently the most compact encoding is to replace pad characters with derivation codes (only prepended not appended). This gives either 1 or 2 character codes. When there are no pad characters then the code length is 3 characters. As a result the CESR code table consists primarily of 1, 2, and 4 character codes. Longer codes may be used as long as they satisfy the padding constraint. Thus for converted cryptographic material with 1 pad character, the allowed code lengths are 1, 5, 9, ... characters. For converted cryptographic material with 2 pad characters the allowed code lengths are 2, 6, 10, ... characters. For converted cryptographic material with 0 pad characters the allowed code lengths are 4, 8, 12, ... characters.





https://datatracker.ietf.org/doc/html/rfc4648

https://multiformats.io/multihash/

https://en.bitcoin.it/wiki/Base58Check_encoding
https://en.bitcoin.it/wiki/Wallet_import_format

https://en.wikipedia.org/wiki/Binary-to-text_encoding

https://stomp.github.io
https://assets.ctfassets.net/3prze68gbwl1/5fKNVB8OWEC1pr7h96jnO3/d1ee79e160398554893158370269839c/overview-of-realtime-streaming-protocols.pdf

Core affinity
https://crd.lbl.gov/assets/Uploads/Nathan-NDM14.pdf

Base64URL File Safe Standard RFC 4648  

https://tools.ietf.org/html/rfc4648  

JSON data model [RFC4627].  

UTF-8 Standard  

https://en.wikipedia.org/wiki/UTF-8  

ASCII Standard  

https://en.wikipedia.org/wiki/ASCII  

IPFS Multihash  

https://richardschneider.github.io/net-ipfs-core/api/Ipfs.Registry.HashingAlgorithm.html  

IPFS Multicodec Multihash Hash Algorithm Table  

 https://github.com/multiformats/multicodec/blob/master/table.csv  

Multicodec   

 https://github.com/multiformats/multicodec/blob/master/README.md  


JSON Mapping Object Delimeters  

https://www.json.org/json-en.html  

CBOR Mapping Object Codes  

https://en.wikipedia.org/wiki/CBOR  

MsgPack Mapping Object Codes  

https://github.com/msgpack/msgpack/blob/master/spec.md  

UTF Byte Order Mark (BOM)  

https://en.wikipedia.org/wiki/Byte_order_mark  

### Background

These codes are also represented in the textual domain with characters drawn from the Base64 set of characters. Namely Base64 uses characters [A-Z,a-z,-,_]. In addition the "=" character is used as a pad character to ensure lossless round tripping of concatenated binary items to Base64 and back. These 64 characters map to the values [0-63].

But simply, each 8 bit long Base64 character represents only 6 bits of information. Each binary byte represents a full 8 bits of information. 



CESR derivation codes serve several purposes and provide several features. One main purpose is to allow compact encoding of cryptographic material in the text domain. What we mean by text domain is any representation that is is restricted to printable/viewable ASCII text characters. Cryptographic material is largely composed of long strings of pseudo-random numbers. CESR uses the IETF RFC-3638 Base64URL standard to encode cryptographic material as strings of binary bytes into a text domain representation [[1]]. In order to process a given cryptographic material item its derivation from other cryptographic material and the cryptographic suite of operations that govern that derivation also need to be known. Typically this additional cryptographic information may be provided via some datastructure. For compactness CESR encodes the derivation information which includes the cryptographic suite into a lookup table of codes. To keep the codes short, only the essential information is encoded in the table. CESR MUST be used in context so that code and context togetherr fully characterize the cryptographic material usage and derivation. This also contributes to compactness.

These codes are also represented in the textual domain with characters drawn from the Base64 set of characters. Namely Base64 uses characters [A-Z,a-z,-,_]. In addition the "=" character is used as a pad character to ensure lossless round tripping of concatenated binary items to Base64 and back. These 64 characters map to the values [0-63]. The Base64 derivation code is prepended to a given Base64 converted cryptographic material item to produce an extremely compact text domain representation of a given cryptographic material item. When a Base64 derivation code is prepended to a Base64 encoded cryptographic material item, the resultant string of characters is called fully qualified Base64 or qualified Base64 and may be labeled "qb64" for short. We call this a fully qualified cryptographic primitive. This fully qualified compact string (a primitive) may then be used in any text domain representation, including especially name spaces. One other less obvious but important property of KERI's encoding is that all qualified cryptographic material items satisfy what we call lossless composition via concatenation, or composabilty for short. KERI is designed for high performance asynchronous data streaming and event sourcing applications. The sheer volume of cryptographic material, primarily, signatures, in a streaming application demands a streamable protocol. Many of KERI's important use cases benefit specifically from streaming in the text domain not merely the binary domain. Composability allows text domain streams of primitives or portions of streams (streamlets) to be converted as a whole to the binary domain and back again without loss. The attached KID0001 Comment document goes into some length on what composability means. 

But simply, each 8 bit long Base64 character represents only 6 bits of information. Each binary byte represents a full 8 bits of information. When converting streams made up of concatenated primitives back and forth between the text and binary domains, the converted results will not align on byte or character boundaries at the end of each primitive unless the primitives themselves are integer multiples of 24 bits of information. Twenty-four is the least common multiple of six and eight. It takes 3 binary bytes to represent 24 bits of information and 4 Base64 characters to represent 24 bits of information. Composability via concatenation is guaranteed if any primitive is an integer multiple of four characters in the text domain and an integer multiple of three bytes in the binary domain. KERI's derivation code table is designed to satisfy this composability constraint. In other words, fully qualified KERI cryptographic primitives are composable via concatenation in both the text (Base64)and binary domains. This provides the ability to create composable streaming protocols that use KERI's coding table that are equally at home in the text and binary domains. Without composability some other means of framing, delimiting, or enveloping cryptographic material items is needed for lossless conversion of group of cryptographic material items as a group between the text and binary domain. Mere concatenation of a group, which is the most compact, is not supported without composable primitives.

The length of a composable Base64 derivation code is a function of the length of the converted cyptographic material. The length of the derivation code plus material must be an even multiple of four bytes. Standard Base64 conversions add pad characters to ensure composability of any converted item [[1]]. The number of pad characters is a function of the length of the item to be converted. If the item's length in bytes is a multiple of three bytes then the converted item's length will be a multiple of four base 64 characters and therefore will not need any pad characters. 



CESR's approach to filling the tables is a first needed first served basis. In addition CESR's requirement that all cryptographic operations maintain at least 128 bits of cryptographic strength precludes the entry of many weak cryptographic suites into the tables, at the least in the compact tables. CESR's compact code table includes only best-of-class cryptographic operations. In 2022 it is expected that NIST will approve standardized post-quantum resistant cryptographic signatures (CESR's digests are already post quantum resistant). At which time the most appropriate siganture suites will be added. Falcon appears to be the leader with open source code already available.

There are three types of codes in the code table. The context of where the material appears determines which type of code to use. The first type of code, called a basic code, is for basic primitives. The minimum code length for basic codes is 1. The second type of code, called an indexed code, is for indexed primitives. Indexed codes find application with cryptographic material that is attached to events. The seminal use case is for attached signatures. Becasue KERI operates asynchronously, attached signatures for multi-signature schemes may be collected asynchronously. The number of attached signatures may vary. The index within the code enables compact indication of the associated public key from the ordered key list in the key state that may be used to verify a given attached signature. Indexed codes may be used for other contexts where an offset or count is helpful. The minimum code length for indexed codes is two.  The third type of code, called a counter code, is for counting the number of characters, primitives or groups in a collection that follows the cod. In other words a counter code provides a count of the number of members of a group or the sum of all the characters for all the primitives in that group.  Counter codes allow framing of cryptographic material for ease of parsing a concatenated stream of primitives or groups of primitives. They are also called composition codes because they allow the composition via concatenation of primitives and groups of primitives. This enables truly powerful expressions of parsable composable streaming content in both the text and binary domain. The minimun length of a counter code is four characters. Counter codes are standalone and do not have attached cryptographic material. They are primitives in their own right. Counter codes may be stacked, i.e. multiple counter codes may appear in succession. They perform nested compositions on the following crypto material.



### Base64 Master Code Table

This master table includes all three types of codes separated by headers. The table has 5 columns. These are as follows: 

1) The Base64 code itself. 
2) A description of what is encoded or appended to the code.
3) The length in characters of the code.
4) the length in characters of the index or count portion of the code 
5) The length in characters of the fully qualified primitive including code and append material or number of elements in group. 

For each code type, a special character is used when the code length is longer than the minimum length. For basic codes the minimum length is one. Two characters codes start with "0". Four character codes start with "1". For indexed codes the minimum code length is two.  Four character codes start with "0". For counter codes the minimum code length is four. Counter codes all start with "-" and eight character counter codes start with "-0". 

Parsing operation is as follows: Based on context, an expected code type is selected. The parser then reads the first character of the code. Based on the first character, the parser is able to determine the length of the code. The parser then extracts the full code given its length and then looks up the code in the table to confirm it is a valid code. The table includes the total length of the appended primitive, if any, which allows the parser to then extract the appended characters and perform any conversions if needed on the appended material. For indexed or count codes the table includes the number of characters in the code that are part of the index or count. Lookup in the indexed and count code tables for valid codes do not include the index characters.

The counter codes are special, they enable framing of a group of crypto material primitives or groups of primitives. This can be used by a parser to perform concurrent or pipelined processing of streams of concatenated fully qualified cypto material or to determine the end of attached material after one event and the start of a new event. 

There is a dual representation of the table in the binary domain. A Base64 primitive when converted as a whole from Base64 to Base2 binary is in a format called fully qualified base2 or qualified base2 or qb2 for short. Each code character in qb64 converts to a sextet (6 bits) (two octal digits) in qb2 and the appended crypto material is shifted 2 bits for each pad character equivalent. A parser needs to extract the first sextet to determine how many remaining sextets of code there are and from that look up how many bytes total to extract. A binary domain parser of qb2 format must perform some bit level extraction and shifting operations. But this is not difficult for binary domain parsers. Whereas a text domain parser need only use character level operations. This is described in more detail in the KID0001 Commentary document.  





|   Code   | Description                                                                                       | Code Length | Index Length | Total Length |
|:--------:|---------------------------------------------------------------------------------------------------|-------------|--------------|--------------|
|          |                              **Basic One Character Codes**                                     |             |              |              |
|     A    | Random seed of Ed25519 private key of length 256 bits                                             |      1      |              |      44      |
|     B    | Ed25519 non-transferable prefix public signing verification key. Basic derivation.                |      1      |              |      44      |
|     C    | X25519 public encryption key. May be converted from Ed25519 public signing verification key.      |      1      |              |      44      |
|     D    | Ed25519 public signing verification key. Basic derivation.                                        |      1      |              |      44      |
|     E    | Blake3-256 Digest. Self-addressing derivation.                                                    |      1      |              |      44      |
|     F    | Blake2b-256 Digest. Self-addressing derivation.                                                   |      1      |              |      44      |
|     G    | Blake2s-256 Digest. Self-addressing derivation.                                                   |      1      |              |      44      |
|     H    | SHA3-256 Digest. Self-addressing derivation.                                                      |      1      |              |      44      |
|     I    | SHA2-256 Digest. Self-addressing derivation.                                                      |      1      |              |      44      |
|     J    | Random seed of ECDSA secp256k1 private key of length 256 bits                                     |      1      |              |      44      |
|     K    | Random seed of Ed448 private key of length 448 bits                                               |      1      |              |      76      |
|     L    | X448 public encryption key. May be converted from Ed448 public signing verification key.          |      1      |              |      76      |
|     M    | Short value of length 16 bits                                                                     |      1      |              |       4      |
|          |                              **Basic Two Character Codes**                                     |             |              |              |
|    0A    | Random salt, seed, private key, or sequence number of length 128 bits                             |      2      |              |      24      |
|    0B    | Ed25519 signature. Self-signing derivation.                                                       |      2      |              |      88      |
|    0C    | ECDSA secp256k1 signature. Self-signing derivation.                                               |      2      |              |      88      |
|    0D    | Blake3-512 Digest. Self-addressing derivation.                                                    |      2      |              |      88      |
|    0E    | Blake2b-512 Digest. Self-addressing derivation.                                                   |      2      |              |      88      |
|    0F    | SHA3-512 Digest. Self-addressing derivation.                                                      |      2      |              |      88      |
|    0G    | SHA2-512 Digest. Self-addressing derivation.                                                      |      2      |              |      88      |
|    0H    | Long value of length 32 bits                                                                      |      2      |              |       8      |
|          |                 **Basic Four Character Codes**                                                 |             |              |              |
|   1AAA   | ECDSA secp256k1 non-transferable prefix public signing verification key. Basic derivation.        |      4      |              |      48      |
|   1AAB   | ECDSA secp256k1 public signing verification or encryption key. Basic derivation.                  |      4      |              |      48      |
|   1AAC   | Ed448 non-transferable prefix public signing verification key. Basic derivation.                  |      4      |              |      80      |
|   1AAD   | Ed448 public signing verification key. Basic derivation.                                          |      4      |              |      80      |
|   1AAE   | Ed448 signature. Self-signing derivation.                                                         |      4      |              |      156     |
|   1AAF   | Tag Base64 4 chars or 3 byte number                                                               |      4      |              |      8       |
|   1AAG   | DateTime Base64 custom encoded 32 char ISO-8601 DateTime                                          |      4      |              |      36      |
|          |                          **Indexed Two Character Codes**                                       |             |              |              |
|    A#    | Ed25519 indexed signature                                                                         |      2      |       1      |      88      |
|    B#    | ECDSA secp256k1 indexed signature                                                                 |      2      |       1      |      88      |
|          |                        **Indexed Four Character Codes**                                        |             |              |              |
|   0A##   | Ed448 indexed signature                                                                           |      4      |       2      |      156     |
|   0B##   | Label Base64 chars of variable length L=N*4 where N is value of index  total = L+4                |      4      |       2      |   Variable   |
|          |                        **Counter Four Character Codes**                                        |             |              |              |
|   -A##   | Count of attached qualified Base64 indexed controller signatures                                  |      4      |       2      |       4      |
|   -B##   | Count of attached qualified Base64 indexed witness signatures                                     |      4      |       2      |       4      |
|   -C##   | Count of attached qualified Base64 nontransferable identifier receipt couples  pre+sig            |      4      |       2      |       4      |
|   -D##   | Count of attached qualified Base64 transferable identifier receipt quadruples  pre+snu+dig+sig    |      4      |       2      |       4      |
|   -E##   | Count of attached qualified Base64 first seen replay couples fn+dt                                |      4      |       2      |       4      |
|   -F##   | Count of attached qualified Base64 transferable indexed sig groups pre+snu+dig + idx sig group    |      4      |       2      |       4      |
|          |                                                                                                   |             |              |              |
|   -U##   | Count of qualified Base64 groups or primitives in message data                                    |      4      |       2      |       4      |
|   -V##   | Count of total attached grouped material qualified Base64 4 char quadlets                         |      4      |       2      |       4      |
|   -W##   | Count of total message data grouped material qualified Base64 4 char quadlets                     |      4      |       2      |       4      |
|   -X##   | Count of total group message data plus attachments qualified Base64 4 char quadlets               |      4      |       2      |       4      |
|   -Y##   | Count of qualified Base64 groups or primitives in group. (context dependent)                      |      4      |       2      |       4      |
|   -Z##   | Count of grouped material qualified Base64 4 char quadlets (context dependent)                    |      4      |       2      |       4      |
|          |                                                                                                   |             |              |              |
|   -a##   | Count of anchor seal groups in list  (anchor seal list) (a)                                       |      4      |       2      |       4      |
|   -c##   | Count of config traits (each trait is 4 char quadlet   (configuration trait list) (c)             |      4      |       2      |       4      |
|   -d##   | Count of digest seal Base64 4 char quadlets in digest  (digest seal  (d)                          |      4      |       2      |       4      |
|   -e##   | Count of event seal Base64 4 char quadlets in seal triple of (event seal) (i, s, d)               |      4      |       2      |       4      |
|   -k##   | Count of keys in list  (key list) (k)                                                             |      4      |       2      |       4      |
|   -l##   | Count of locations seal Base64 4 char quadlets in seal quadruple of (location seal) (i, s, t, p)  |      4      |       2      |       4      |
|   -r##   | Count of root digest seal Base64 4 char quadlets in root digest  (root digest) (rd)               |      4      |       2      |       4      |
|   -w##   | Count of witnesses in list  (witness list or witness remove list or witness add list) (w, wr, wa) |      4      |       2      |       4      |
|          |                       **Counter Eight Character Codes**                                        |             |              |              |
| -0U##### | Count of qualified Base64 groups or primitives in message data                                    |      8      |       5      |       8      |
| -0V##### | Count of total attached grouped material qualified Base64 4 char quadlets                         |      8      |       5      |       8      |
| -0W##### | Count of total message data grouped material qualified Base64 4 char quadlets                     |      8      |       5      |       8      |
| -0X##### | Count of total group message data plus attachments qualified Base64 4 char quadlets               |      8      |       5      |       8      |
| -0Y##### | Count of qualified Base64 groups or primitives in group (context dependent)                       |      8      |       5      |       8      |
| -0Z##### | Count of grouped  material qualified Base64 4 char quadlets (context dependent)                   |      8      |       5      |       8      |
|          |                                                                                                   |             |              |              |
| -0a##### | Count of anchor seals  (seal groups in list)                                                      |      8      |       5      |       8      |



The table includes one complex group that is composed of two groups. This is the counter attachment group 
with code`-F##` where ``##`` is replaced by the two character Base64 count of the number of complex groups.  
This is known as the TransIndexedSigGroups counter.  Within the complex group are one more more attached
groups where each group consists of a triple pre+snu+dig
followed by a ControllerIdxSigs group that in turn consists of a counter code `-A##` followed by one or more
indexed signature primitives. The following example details how this complex group may appear.

The example has only one group. The example is annotated with comments, spaces and line feeds for clarity.

~~~
-FAB     # Trans Indexed Sig Groups counter code 1 following group
E_T2_p83_gRSuAYvGhqV3S0JzYEF2dIa-OCPLbIhBO7Y    # trans prefix of signer for sigs
-EAB0AAAAAAAAAAAAAAAAAAAAAAB    # sequence number of est event of signer's public keys for sigs
EwmQtlcszNoEIDfqD-Zih3N6o5B3humRKvBBln2juTEM      # digest of est event of signer's public keys for sigs
-AAD     # Controller Indexed Sigs counter code 3 following sigs
AA5267UlFg1jHee4Dauht77SzGl8WUC_0oimYG5If3SdIOSzWM8Qs9SFajAilQcozXJVnbkY5stG_K4NbKdNB4AQ         # sig 0
ABBgeqntZW3Gu4HL0h3odYz6LaZ_SMfmITL-Btoq_7OZFe3L16jmOe49Ur108wH7mnBaq2E_0U0N0c5vgrJtDpAQ    # sig 1
ACTD7NDX93ZGTkZBBuSeSGsAQ7u0hngpNTZTK_Um7rUZGnLRNJvo5oOnnC1J2iBQHuxoq8PyjdT3BHS2LiPrs2Cg  # sig 2

~~~

Given this comples attachment group we no longer need to embed an Event Seal in messages to indicate the
establishment event for of a transferable signer as is the case for the `vrc` receipt message and as was
origianlly proposed for the `ksn` message. The seal equivalent is provided in the pre+snu+dig or (i,s,d) 
triple in the front of the new complex attachment group.





# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

The keripy development team, the KERI community and the ToIP ACDC working group.
