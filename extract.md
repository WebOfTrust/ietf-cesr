
# Appendix: Master Code Table

## Filling Code Table
The approach to filling the tables is a first needed first-served basis. In addition, the requirement that all cryptographic operations maintain at least 128 bits of cryptographic strength precludes the entry of many weak cryptographic suites into the compact tables. CESR's compact code table includes only best-of-class cryptographic operations. In 2022 it is expected that NIST will approve standardized post-quantum resistant cryptographic signatures at which time codes for the most appropriate post-quantum signature suites will be added. Falcon appears to be the leader with open source code already available.


## Description
This master table includes all three types of codes separated by headers. The table has 5 columns. These are as follows:

1) The Base64 stable (hard) text code itself.
2) A description of what is encoded or appended to the code.
3) The length in characters of the code.
4) the length in characters of the index or count portion of the code
5) The length in characters of the fully qualified primitive including code and append material or number of elements in the group.



|   Code   | Description | Code Length | Count or Index Length | Total Length |
|:--------:|:----------------------------------|:------------:|:-------------:|:------------:|
|          |                    **Basic One Character Codes**                                     |             |              |              |
|     `A`    | Random seed of Ed25519 private key of length 256 bits                                             |      1      |              |      44      |
|     `B`    | Ed25519 non-transferable prefix public signing verification key. Basic derivation.                |      1      |              |      44      |
|     `C`    | X25519 public encryption key. May be converted from Ed25519 public signing verification key.      |      1      |              |      44      |
|     `D`    | Ed25519 public signing verification key. Basic derivation.                                        |      1      |              |      44      |
|     `E`    | Blake3-256 Digest. Self-addressing derivation.                                                    |      1      |              |      44      |
|     `F`    | Blake2b-256 Digest. Self-addressing derivation.                                                   |      1      |              |      44      |
|     `G`    | Blake2s-256 Digest. Self-addressing derivation.                                                   |      1      |              |      44      |
|     `H`    | SHA3-256 Digest. Self-addressing derivation.                                                      |      1      |              |      44      |
|     `I`    | SHA2-256 Digest. Self-addressing derivation.                                                      |      1      |              |      44      |
|     `J`    | Random seed of ECDSA secp256k1 private key of length 256 bits                                     |      1      |              |      44      |
|     `K`    | Random seed of Ed448 private key of length 448 bits                                               |      1      |              |      76      |
|     `L`    | X448 public encryption key. May be converted from Ed448 public signing verification key.          |      1      |              |      76      |
|     `M`    | Short value of length 16 bits                                                                     |      1      |              |       4      |
|          |                **Basic Two Character Codes**                                     |             |              |              |
|    `0A`    | Random salt, seed, private key, or sequence number of length 128 bits                             |      2      |              |      24      |
|    `0B`    | Ed25519 signature. Self-signing derivation.                                                       |      2      |              |      88      |
|    `0C`    | ECDSA secp256k1 signature. Self-signing derivation.                                               |      2      |              |      88      |
|    `0D`    | Blake3-512 Digest. Self-addressing derivation.                                                    |      2      |              |      88      |
|    `0E`    | Blake2b-512 Digest. Self-addressing derivation.                                                   |      2      |              |      88      |
|    `0F`    | SHA3-512 Digest. Self-addressing derivation.                                                      |      2      |              |      88      |
|    `0G`    | SHA2-512 Digest. Self-addressing derivation.                                                      |      2      |              |      88      |
|    `0H`    | Long value of length 32 bits                                                                      |      2      |              |       8      |
|          |                 **Basic Four Character Codes**                                      |             |              |              |
|   `1AAA`   | ECDSA secp256k1 non-transferable prefix public signing verification key. Basic derivation.        |      4      |              |      48      |
|   `1AAB`   | ECDSA secp256k1 public signing verification or encryption key. Basic derivation.                  |      4      |              |      48      |
|   `1AAC`   | Ed448 non-transferable prefix public signing verification key. Basic derivation.                  |      4      |              |      80      |
|   `1AAD`   | Ed448 public signing verification key. Basic derivation.                                          |      4      |              |      80      |
|   `1AAE`   | Ed448 signature. Self-signing derivation.                                                         |      4      |              |      156     |
|   `1AAF`   | Tag Base64 4 chars or 3 byte number                                                               |      4      |              |      8       |
|   `1AAG`   | DateTime Base64 custom encoded 32 char ISO-8601 DateTime                                          |      4      |              |      36      |
|          |                          **Indexed Two Character Codes**                           |             |              |              |
|    `A#`    | Ed25519 indexed signature                                                                         |      2      |       1      |      88      |
|    `B#`    | ECDSA secp256k1 indexed signature                                                                 |      2      |       1      |      88      |
|          |                        **Indexed Four Character Codes**                          |             |              |              |
|   `0A##`   | Ed448 indexed signature                                                                           |      4      |       2      |      156     |
|   `0B##`   | Label Base64 chars of variable length `L=N*4` where N is value of index                |      4      |       2      |   Variable   |
|          |                        **Counter Four Character Codes**                           |             |              |              |
|   `-A##`   | Count of attached qualified Base64 indexed controller signatures                                  |      4      |       2      |       4      |
|   `-B##`   | Count of attached qualified Base64 indexed witness signatures                                     |      4      |       2      |       4      |
|   `-C##`   | Count of attached qualified Base64 nontransferable identifier receipt couples  pre+sig            |      4      |       2      |       4      |
|   `-D##`   | Count of attached qualified Base64 transferable identifier receipt quadruples  pre+snu+dig+sig    |      4      |       2      |       4      |
|   `-E##`   | Count of attached qualified Base64 first seen replay couples fn+dt                                |      4      |       2      |       4      |
|   `-F##`   | Count of attached qualified Base64 transferable indexed sig groups pre+snu+dig + idx sig group    |      4      |       2      |       4      |
|          |                                                                                 |             |              |              |
|   `-U##`   | Count of qualified Base64 groups or primitives in message data                                    |      4      |       2      |       4      |
|   `-V##`   | Count of total attached grouped material qualified Base64 4 char quadlets                         |      4      |       2      |       4      |
|   `-W##`   | Count of total message data grouped material qualified Base64 4 char quadlets                     |      4      |       2      |       4      |
|   `-X##`   | Count of total group message data plus attachments qualified Base64 4 char quadlets               |      4      |       2      |       4      |
|   `-Y##`   | Count of qualified Base64 groups or primitives in group. (context dependent)                      |      4      |       2      |       4      |
|   `-Z##`   | Count of grouped material qualified Base64 4 char quadlets (context dependent)                    |      4      |       2      |       4      |
|          |                                                                                                   |             |              |              |
|   `-a##`   | Count of anchor seal groups in list  (anchor seal list) (a)                                       |      4      |       2      |       4      |
|   `-c##`   | Count of config traits (each trait is 4 char quadlet   (configuration trait list) (c)             |      4      |       2      |       4      |
|   `-d##`   | Count of digest seal Base64 4 char quadlets in digest  (digest seal  (d)                          |      4      |       2      |       4      |
|   `-e##`   | Count of event seal Base64 4 char quadlets in seal triple of (event seal) (i, s, d)               |      4      |       2      |       4      |
|   `-k##`   | Count of keys in list  (key list) (k)                                                             |      4      |       2      |       4      |
|   `-l##`   | Count of locations seal Base64 4 char quadlets in seal quadruple of (location seal) (i, s, t, p)  |      4      |       2      |       4      |
|   `-r##`   | Count of root digest seal Base64 4 char quadlets in root digest  (root digest) (rd)               |      4      |       2      |       4      |
|   `-w##`   | Count of witnesses in list  (witness list or witness remove list or witness add list) (w, wr, wa) |      4      |       2      |       4      |
|          |                       **Counter Eight Character Codes**                             |             |              |              |
| `-0U#####` | Count of qualified Base64 groups or primitives in message data                                    |      8      |       5      |       8      |
| `-0V#####` | Count of total attached grouped material qualified Base64 4 char quadlets                         |      8      |       5      |       8      |
| `-0W#####` | Count of total message data grouped material qualified Base64 4 char quadlets                     |      8      |       5      |       8      |
| `-0X#####` | Count of total group message data plus attachments qualified Base64 4 char quadlets               |      8      |       5      |       8      |
| `-0Y#####` | Count of qualified Base64 groups or primitives in group (context dependent)                       |      8      |       5      |       8      |
| `-0Z#####` | Count of grouped  material qualified Base64 4 char quadlets (context dependent)                   |      8      |       5      |       8      |
|          |                                                                      |             |              |              |
| `-0a#####` | Count of anchor seals  (seal groups in list)                                                      |      8      |       5      |       8      |



The table includes complex groups that are composed of other groups. For example, consider the counter attachment group
with code `-F##` where `##` is replaced by the two-character Base64 count of the number of complex groups.
This is known as the TransIndexedSigGroups counter.  Within the complex group are one or more attached
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



