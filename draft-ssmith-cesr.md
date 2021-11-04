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


tags: IETF, SAID, ACDC

--- abstract

TODO Abstract


--- middle

# Introduction

CESR derivation codes serve several purposes and provide several features. One main purpose is to allow compact encoding of cryptographic material in the text domain. What we mean by text domain is any representation that is restricted to printable/viewable ASCII text characters. Cryptographic material is largely composed of long strings of pseudo-random numbers. CESR uses the IETF RFC-3638 Base64URL standard to encode cryptographic material as strings of binary bytes into a text domain representation [[1]]. In order to process a given cryptographic material item, its derivation from other cryptographic material and the cryptographic suite of operations that govern that derivation also need to be known. Typically this additional cryptographic information may be provided via some datastructure. For compactness CESR encodes the derivation information which includes the cryptographic suite into a lookup table of codes. To keep the codes short, only the essential information is encoded in the table. CESR MUST be used in context so that code and context together fully characterize the cryptographic material usage and derivation. This also contributes to compactness.

These codes are also represented in the textual domain with characters drawn from the Base64 set of characters. Namely Base64 uses characters [A-Z,a-z,-,_]. In addition the "=" character is used as a pad character to ensure lossless round tripping of concatenated binary items to Base64 and back. These 64 characters map to the values [0-63]. The Base64 derivation code is prepended to a given Base64 converted cryptographic material item to produce an extremely compact text domain representation of a given cryptographic material item. When a Base64 derivation code is prepended to a Base64 encoded cryptographic material item, the resultant string of characters is called fully qualified Base64 or qualified Base64 and may be labeled "qb64" for short. We call this a fully qualified cryptographic primitive. This fully qualified compact string (a primitive) may then be used in any text domain representation, including especially name spaces. One other less obvious but important property of KERI's encoding is that all qualified cryptographic material items satisfy what we call lossless composition via concatenation, or composabilty for short. KERI is designed for high performance asynchronous data streaming and event sourcing applications. The sheer volume of cryptographic material, primarily, signatures, in a streaming application demands a streamable protocol. Many of KERI's important use cases benefit specifically from streaming in the text domain not merely the binary domain. Composability allows text domain streams of primitives or portions of streams (streamlets) to be converted as a whole to the binary domain and back again without loss. The attached KID0001 Comment document goes into some length on what composability means. 

But simply, each 8 bit long Base64 character represents only 6 bits of information. Each binary byte represents a full 8 bits of information. When converting streams made up of concatenated primitives back and forth between the text and binary domains, the converted results will not align on byte or character boundaries at the end of each primitive, unless the primitives themselves are integer multiples of 24 bits of information. Twenty-four is the least common multiple of six and eight. It takes 3 binary bytes to represent 24 bits of information and 4 Base64 characters to represent 24 bits of information. Composability via concatenation is guaranteed if any primitive is an integer multiple of four characters in the text domain and an integer multiple of three bytes in the binary domain. KERI's derivation code table is designed to satisfy this composability constraint. In other words, fully qualified KERI cryptographic primitives are composable via concatenation in both the text (Base64)and binary domains. This provides the ability to create composable streaming protocols that use KERI's coding table and are equally at home in the text and binary domains. Without composability some other means of framing, delimiting, or enveloping cryptographic material items is needed for lossless conversion of group of cryptographic material items as a group between the text and binary domain. Mere concatenation of a group, which is the most compact, is not supported without composable primitives.

The length of a composable Base64 derivation code is a function of the length of the converted cyptographic material. The length of the derivation code plus material must be an even multiple of four bytes. Standard Base64 conversions add pad characters to ensure composability of any converted item [[1]]. The number of pad characters is a function of the length of the item to be converted. If the item's length in bytes is a multiple of three bytes then the converted item's length will be a multiple of four base 64 characters and therefore will not need any pad characters. Any binary item whose length mod 3 is 1 will need 2 pad characters to makes its converted length a multiple of four characters. Any binary item whose length mod 3 is 2 will need 1 pad characters to makes its converted length a multiple of four characters. Consequently the most compact encoding is to replace pad characters with derivation codes (only prepended not appended). This gives either 1 or 2 character codes. When there are no pad characters then the code length is 3 characters. As a result the CESR code table consists primarily of 1, 2, and 4 character codes. Longer codes may be used as long as they satisfy the padding constraint. Thus for converted cryptographic material with 1 pad character, the allowed code lengths are 1, 5, 9, ... characters. For converted cryptographic material with 2 pad characters the allowed code lengths are 2, 6, 10, ... characters. For converted cryptographic material with 0 pad characters the allowed code lengths are 4, 8, 12, ... characters.

CESR's approach to filling the tables is a first needed, first served basis. In addition, CESR's requirement that all cryptographic operations maintain at least 128 bits of cryptographic strength, precludes the entry of many weak cryptographic suites into the tables, at the least in the compact tables. CESR's compact code table includes only best-of-class cryptographic operations. In 2022 it is expected that NIST will approve standardized post-quantum resistant cryptographic signatures (CESR's digests are already post-quantum resistant). At which time the most appropriate signature suites will be added. Falcon appears to be the leader with open source code already available.

There are three types of codes in the code table. The context of where the material appears determines which type of code to use. The first type of code, called a basic code, is for basic primitives. The minimum code length for basic codes is 1. The second type of code, called an indexed code, is for indexed primitives. Indexed codes find application with cryptographic material that is attached to events. The seminal use case is for attached signatures. Becasue KERI operates asynchronously, attached signatures for multi-signature schemes may be collected asynchronously. The number of attached signatures may vary. The index within the code enables compact indication of the associated public key from the ordered key list in the key state that may be used to verify a given attached signature. Indexed codes may be used for other contexts where an offset or count is helpful. The minimum code length for indexed codes is two. The third type of code, called a counter code, is for counting the number of characters, primitives or groups in a collection that follows the code. In other words, a counter code provides a count of the number of members of a group or the sum of all the characters for all the primitives in that group.  Counter codes allow framing of cryptographic material for ease of parsing a concatenated stream of primitives or groups of primitives. They are also called composition codes because they allow the composition via concatenation of primitives and groups of primitives. This enables truly powerful expressions of parsable composable streaming content in both the text and binary domain. The minimun length of a counter code is four characters. Counter codes are standalone and do not have attached cryptographic material. They are primitives in their own right. Counter codes may be stacked, i.e. multiple counter codes may appear in succession. They perform nested compositions on the following crypto material.



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
This is known as the TransIndexedSigGroups counter.  Within the complex group there are one or more attached
groups, where each group consists of a triple pre+snu+dig
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

Given this complex attachment group, we no longer need to embed an Event Seal in messages to indicate the
establishment event for a transferable signer, as is the case for the `vrc` receipt message and as was
originally proposed for the `ksn` message. The seal equivalent is provided in the pre+snu+dig or (i,s,d) 
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
