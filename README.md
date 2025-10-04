Caret-Encoding
==============

Caret-Encoding is an escape-based encoding scheme that allows Unicode codepoints to be represented numerically in cases where the actual codepoint itself cannot be used for some reason.

This could be due to:

- Missing or incomplete Unicode support
- Codepoints that are restricted in a particular medium
- Representing a codepoint that is already being used as a delimiter

Caret-Encoding's primary use case is for representing arbitrary codepoints in file names, but it could also be used in other mediums that restrict otherwise allowable characters.


----------------------------------------------------------------------------------------------------

Terms and Conventions
---------------------

**The following bolded, capitalized terms have specific meanings in this document**:

| Term             | Meaning                                                                       |
|------------------|-------------------------------------------------------------------------------|
| **MUST (NOT)**   | If this directive is not adhered to, the implementation is invalid.           |
| **CAN**          | Refers to a possibility which **MUST** be accommodated by the implementation. |

----------------------------------------------------------------------------------------------------


Encoding
--------

A caret-encoded Unicode codepoint is represented as a character sequence beginning with the caret (`^`) character, followed by an optional modifier character, and finally a sequence of hexadecimal digits (`0`-`9`, `a`-`f`, `A`-`F`) representing the codepoint.

If no modifier character is present, only two hexadecimal digits follow; otherwise more hexadecimal digits follow according to the following table:

| Initiator | Modifier   | Digit Count | Example                         |
|-----------|------------|-------------|---------------------------------|
| `^`       |            | 2           | `^2F` (`/`)                     |
| `^`       | `g` or `G` | 3           | `^g141` (`Ł`)                   |
| `^`       | `h` or `H` | 4           | `^h2021` (`‡`)                  |
| `^`       | `i` or `I` | 5           | `^i1D120` (`𝄠`)                 |
| `^`       | `j` or `J` | 6           | `^j10ABCD` (user-defined value) |

### Encoding Rules

- Any Unicode codepoint **CAN** be encoded.
- All codepoints from u+0000 to u+001F, and u+007F (control characters) **MUST** be encoded.
- Most non-alphanumeric characters below codepoint u+007F **MUST** be encoded because they have special meanings in various systems, or are problematic in certain circumstances.

Below is a table of all non-alphanumeric characters from codepoint u+0020 to u+007E:

| Codepoint | Symbol | Issue       | Context                                             |
|-----------|--------|-------------|-----------------------------------------------------|
|  u+0020   |  ` `   | Problematic | Windows doesn't allow leading or trailing spaces    |
|  u+0021   |   !    | Reserved    | URI                                                 |
|  u+0022   |   "    | Reserved    | Windows FS, IBM FS                                  |
|  u+0023   |   #    | Reserved    | URI, Sharepoint, problematic for IBM FS             |
|  u+0024   |   $    | Reserved    | URI                                                 |
|  u+0025   |   %    | Reserved    | Sharepoint, Percent-Encoding                        |
|  u+0026   |   &    | Reserved    | URI, Various Microsoft products                     |
|  u+0027   |   '    | Reserved    | URI                                                 |
|  u+0028   |   (    | Reserved    | URI                                                 |
|  u+0029   |   )    | Reserved    | URI                                                 |
|  u+002a   |   *    | Reserved    | URI, Windows FS, IBM FS                             |
|  u+002b   |   +    | Reserved    | URI                                                 |
|  u+002c   |   ,    | Reserved    | URI                                                 |
|  u+002d   |   -    | Safe        |                                                     |
|  u+002e   |   .    | Problematic | Windows doesn't allow a directory to end with `.`   |
|  u+002f   |   /    | Reserved    | URI, Windows FS, Unix FS                            |
|  u+003a   |   :    | Reserved    | URI, Windows FS                                     |
|  u+003b   |   ;    | Reserved    | URI, problematic in Sharepoint                      |
|  u+003c   |   <    | Reserved    | Windows FS, IBM FS                                  |
|  u+003d   |   =    | Reserved    | URI                                                 |
|  u+003e   |   >    | Reserved    | Windows FS, IBM FS                                  |
|  u+003f   |   ?    | Reserved    | URI, Windows FS, IBM FS                             |
|  u+0040   |   @    | Reserved    | URI                                                 |
|  u+005b   |   [    | Reserved    | URI                                                 |
|  u+005c   |   \    | Reserved    | Windows FS, IBM FS                                  |
|  u+005d   |   ]    | Reserved    | URI                                                 |
|  u+005e   |   ^    | Initiator   | Caret-Encoding                                      |
|  u+005f   |   _    | Safe        |                                                     |
|  u+0060   |   `    | Safe        |                                                     |
|  u+007b   |   {    | Safe        |                                                     |
|  u+007c   |   \|   | Reserved    | Windows FS                                          |
|  u+007d   |   }    | Safe        |                                                     |
|  u+007e   |   ~    | Problematic | Files beginning with `~$` are restricted on Windows |

- Codepoints marked "Reserved" **MUST** be encoded.
- Codepoints marked "Problematic" **MUST** be encoded in the problematic situation.
- The caret initiator (`^`) **MUST** be encoded.
- Codepoints marked "Safe" don't have to be encoded.

**Note**: Since URIs can contain filenames, URI symbol restrictions are also applied in order to promote seamless interoperability.

### Examples

* `^h5927^h5207^h306A^h30D5^h30A1^h30A4^h30EB.doc` (`大切なファイル.doc`)
* `^22secret^22-data.bin` (`"secret"-data.bin`)
* `^i1F607.txt` (`😇.txt`)




Why not use Percent-Encoding instead?
-------------------------------------

[Percent-Encoding](https://datatracker.ietf.org/doc/html/rfc3986#page-12) is designed specifically for encoding data into a URI. Although it could in theory be applied to filenames, there are problems:

- Percent-Encoding doesn't restrict some symbols that are problematic in filenames.
- Percent-Encoding is limited to octets rather than codepoints, which has led to ambiguouity and incompatibilities among implementations.
- Percent-encoded filenames would generate even more bloat when represented as URIs because the escape character would itself have to be escaped:

| Format                 | Representation                                                   |
|------------------------|------------------------------------------------------------------|
| Logical File Name      | `https://example.org/index.html`                                 |
| Caret-Encoded          | `https^3A^2F^2Fexample.org^2Findex.html`                         |
| Percent-Encoded        | `https^%A%2F%2Fexample.org%2Findex.html`                         |
| Caret-Encoded in URI   | `file:///path/to/https^3A^2F^2Fexample.org^2Findex.html`         |
| Percent-Encoded in URI | `file:///path/to/https%253A%252F%252Fexample.org%252Findex.html` |

Also, a separate filename encoding makes the intent clear.


Formal Grammar
--------------

```dogma
dogma_v1 utf-8
- identifier  = caret-encoding
- description = Unicode encoding for file names
- reference   = https://github.com/kstenerud/caret-encoding
- dogma       = https://github.com/kstenerud/dogma/blob/master/v1/dogma_v1.0.md

encoded_codepoint = initiator
                  & var(type, (G | H | I | J)?)
                  & hex_digit{2}
                  & [
                         type = G: hex_digit{1};
                         type = H: hex_digit{2};
                         type = I: hex_digit{3};
                         type = J: hex_digit{4};
                  ];

initiator         = '^';
G                 = 'g' | 'G';
H                 = 'h' | 'H';
I                 = 'i' | 'I';
J                 = 'j' | 'J';
hex_digit         = '0'~'9' | 'a'~'f' | 'A'~'F';
```


References
----------

- [Wikipedia: Problematic filename characters](https://en.wikipedia.org/wiki/Filename#Problematic_characters)
- [Microsoft: Filename restrictions](https://support.microsoft.com/en-us/office/restrictions-and-limitations-in-onedrive-and-sharepoint-64883a5d-228e-48f5-b3d2-eb39e07630fa)
- [IBM: Volume, directory, and file names](https://www.ibm.com/docs/en/i/7.6.0?topic=format-volume-directory-file-names)
- [RFC 3986: Percent-Encoding](https://datatracker.ietf.org/doc/html/rfc3986#page-12)


License
-------

Copyright (c) 2025 Karl Stenerud. All rights reserved.

Distributed under the [Creative Commons Attribution License](https://creativecommons.org/licenses/by/4.0/legalcode) ([license deed](https://creativecommons.org/licenses/by/4.0).
