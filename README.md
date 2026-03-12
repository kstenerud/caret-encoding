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

| Term             | Meaning                                                                                          |
| ---------------- | ------------------------------------------------------------------------------------------------ |
| **MUST (NOT)**   | If this directive is not adhered to, the document or implementation is invalid.                  |
| **SHOULD (NOT)** | Every effort should be made to follow this directive, but it's still conformant if not followed. |
| **CAN**          | Refers to a possibility which **MUST** be accommodated by the implementation.                    |

-----------------------------------------------------------------------------------------------------------------------


Encoding
--------

A caret-encoded Unicode codepoint is represented as a character sequence beginning with the caret (`^`) character, followed by either a single shortcut character or a hex sequence. The hex sequence consists of an optional modifier character and a series of hexadecimal digits (`0`-`9`, `a`-`f`, `A`-`F`) representing the codepoint.

#### Hex Sequences

If no modifier character is present, only two hexadecimal digits follow. If a modifier character from `w` to `z` (case-insensitive) is present, more hexadecimal digits follow according to the following table:

| Initiator | Modifier   | Digit Count | Example                         |
|-----------|------------|-------------|---------------------------------|
| `^`       |            | 2           | `^2F` (`/`)                     |
| `^`       | `w` or `W` | 3           | `^w141` (`Ł`)                   |
| `^`       | `x` or `X` | 4           | `^x2021` (`‡`)                  |
| `^`       | `y` or `Y` | 5           | `^y1D120` (`𝄠`)                 |
| `^`       | `z` or `Z` | 6           | `^z10ABCD` (user-defined value) |

#### Shortcuts

Commonly encoded characters have single-character shortcuts to reduce length requirements and improve readability. Both shortcut and hex forms are valid and equivalent. Letter shortcuts are case-insensitive (e.g. `^S` = `^s` = `^2F`).

Decoders **MUST** recognize both shortcut and hex forms. Encoders **CAN** use either form.

| Shortcut | Char | Codepoint | Mnemonic          |
|----------|------|-----------|-------------------|
| `^^`     | `^`  | u+005E    | Self-escape       |
| `^_`     | ` `  | u+0020    | Underscore=space  |
| `^-`     | `=`  | u+003D    | Horizontal line   |
| `` ^` `` | `+`  | u+002B    |                   |
| `^{`     | `(`  | u+0028    | Opening bracket   |
| `^}`     | `)`  | u+0029    | Closing bracket   |
| `^g`     | `>`  | u+003E    | **G**reater than  |
| `^h`     | `#`  | u+0023    | **H**ash          |
| `^i`     | `!`  | u+0021    | Excla**i**m       |
| `^j`     | `'`  | u+0027    |                   |
| `^k`     | `:`  | u+003A    | **K**olon         |
| `^l`     | `<`  | u+003C    | **L**ess than     |
| `^m`     | `%`  | u+0025    | Per**m**ille      |
| `^n`     | `&`  | u+0026    | A**n**d           |
| `^o`     | `@`  | u+0040    | R**o**und-a       |
| `^p`     | `\|` | u+007C    | **P**ipe          |
| `^q`     | `?`  | u+003F    | **Q**uestion      |
| `^r`     | `\`  | u+005C    | **R**everse slash |
| `^s`     | `/`  | u+002F    | **S**lash         |
| `^t`     | `*`  | u+002A    | S**t**ar          |
| `^u`     | `"`  | u+0022    | Q**u**ote         |
| `^v`     | `$`  | u+0024    | **V**alue         |

### Encoding Rules

- Any Unicode codepoint **CAN** be encoded.
- All codepoints from **u+0000** to **u+001F**, and **u+007F** (C0 control characters) **MUST** be encoded.
- All codepoints from **u+0080** to **u+009F** (C1 control characters) **MUST** be encoded.
- All codepoints from **u+D800** to **u+DFFF** (surrogates) **MUST** be encoded.
- All non-characters **MUST** be encoded: **u+FDD0** to **u+FDEF**, and the last two codepoints of each plane (**u+FFFE, u+FFFF, u+1FFFE, u+1FFFF, ... u+10FFFE, u+10FFFF**).
- Most non-alphanumeric characters below codepoint u+007F **MUST** be encoded because they have special meanings in various systems, or are problematic in certain circumstances.

Below is a table of all non-alphanumeric characters from codepoint u+0020 to u+007E:

| Codepoint | Symbol | Issue       | Context                                             |
|-----------|--------|-------------|-----------------------------------------------------|
|  u+0020   |  ` `   | Problematic | Windows FS: Leading and trailing spaces forbidden   |
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
|  u+002e   |   .    | Problematic | Windows FS: Directory name must not end with `.`    |
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
|  u+007e   |   ~    | Problematic | Windows FS: Files beginning with `~$` are forbidden |

- Codepoints marked "Reserved" **MUST** be encoded.
- Codepoints marked "Problematic" **MUST** be encoded in the problematic situation.
- The caret initiator (`^`) **MUST** be encoded.
- Codepoints marked "Safe" don't have to be encoded.

**Note**: Since URIs can contain filenames, URI symbol restrictions are also applied in order to promote seamless interoperability.

#### Higher Unicode Concerns

Bidirectional control characters **SHOULD** be encoded (**security**: deceptive filenames):
- **u+200E** LEFT-TO-RIGHT MARK
- **u+200F** RIGHT-TO-LEFT MARK
- **u+202A-u+202E** (LTR/RTL embedding, override, pop directional formatting)
- **u+2066-u+2069** (LTR/RTL isolate, first strong isolate, pop directional isolate)

Zero-width and invisible format characters **SHOULD** be encoded (**security**: could create visually indistinguishable filenames):
- **u+00AD** SOFT HYPHEN
- **u+200B** ZERO WIDTH SPACE
- **u+200C** ZERO WIDTH NON-JOINER
- **u+200D** ZERO WIDTH JOINER
- **u+2060** WORD JOINER
- **u+FEFF** BOM / ZERO WIDTH NO-BREAK SPACE

Interlinear annotation characters **SHOULD** be encoded:
- **u+FFF9-u+FFFB**

Tag characters **SHOULD** be encoded:
- **u+E0001-u+E007F**

**Note**: The replacement character (u+FFFD) does not require encoding, but its presence in a filename typically indicates a prior encoding error.

### Examples

* `^x5927^x5207^x306A^x30D5^x30A1^x30A4^x30EB.doc` (`大切なファイル.doc`)
* `^usecret^u-data.bin` (`"secret"-data.bin`) (using `^u` shortcut for `"`)
* `^y1F607.txt` (`😇.txt`)
* `https^k^s^sexample.org^sindex.html` (`https://example.org/index.html`) (using shortcuts for `:` and `/`)




Why not use Percent-Encoding instead?
-------------------------------------

[Percent-Encoding](https://datatracker.ietf.org/doc/html/rfc3986#page-12) is designed specifically for encoding data into a URI. Although it could in theory be applied to filenames, there are problems:

- Percent-Encoding doesn't restrict some symbols that are problematic in filenames.
- Percent-Encoding is limited to octets rather than codepoints, which has led to ambiguouity and incompatibilities among implementations.
- Percent-encoded filenames would generate even more bloat when represented as URIs because the escape character would itself have to be escaped:

| Format                 | Representation                                                   |
|------------------------|------------------------------------------------------------------|
| Logical File Name      | `https://example.org/index.html`                                 |
| Caret-Encoded          | `https^k^s^sexample.org^sindex.html`                             |
| Percent-Encoded        | `https^%A%2F%2Fexample.org%2Findex.html`                         |
| Caret-Encoded in URI   | `file:///path/to/https^k^s^sexample.org^sindex.html`             |
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

encoded_codepoint = initiator & (shortcut | hex_sequence);

hex_sequence      = var(type, (W | X | Y | Z)?)
                  & hex_digit{2}
                  & [
                         type = W: hex_digit{1};
                         type = X: hex_digit{2};
                         type = Y: hex_digit{3};
                         type = Z: hex_digit{4};
                  ];

shortcut          = '^' | '_' | '-' | '`' | '{' | '}'
                  | 'g' | 'G' | 'h' | 'H' | 'i' | 'I' | 'j' | 'J'
                  | 'k' | 'K' | 'l' | 'L' | 'm' | 'M' | 'n' | 'N'
                  | 'o' | 'O' | 'p' | 'P' | 'q' | 'Q' | 'r' | 'R'
                  | 's' | 'S' | 't' | 'T' | 'u' | 'U' | 'v' | 'V';

initiator         = '^';
W                 = 'w' | 'W';
X                 = 'x' | 'X';
Y                 = 'y' | 'Y';
Z                 = 'z' | 'Z';
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
