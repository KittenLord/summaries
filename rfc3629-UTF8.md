This one is actually quite succinct, the summary is not necessary

Source: https://datatracker.ietf.org/doc/html/rfc3629

# UTF-8, a transformation format of ISO 10646

## Abstract

UTF-8 is an encoding for a character sed defined by ISO/IEC 10646-1 that's called the Universal Character Set (UCS, Unicode). UTF-8 preserves the full US-ASCII range, and is compatible with US-ASCII reliant things

## 1. Introduction

All stardard encodings except UTF-8 have an encoding unit larget than one octet, so they're not comfortable to use

US-ASCII characters have the same encodings in UTF-8, and are completely unambiguous

UTF-8 encodes UCS characters as a varying number of octets, where the number of octets depends on the value of the character assigned by ISO/IEC 10646. This encoding has the following characteristics:

- Characters from U+0000 to U+007F (US-ASCII) correspond to octets 0x00 to 0x7F. A plain ASCII string is a valid UTF-8 string

- US-ASCII octets do not appear in a UTF-8 string in any other context, apart from their direct representation

- Round-trip conversion is easy between UTF-8 and other encoding forms

- The first octet of a multi-octet sequence indicates the number of octets in the sequence

- The octet values 0xC0, 0xC1, 0xF5..0xFF never appear

- Character boundaries are easily found from anywhere in an octet stream

- The byte-value sorting order of UTF-8 strings is the same as if ordered by character numbers

- The Boyer-Moore fast search algorithm can be used with UTF-8 data

- UTF-8 strings can be reliably detected as such within an arbitrary binary stream

UTF-8 was originally designed for the Plan9 OS by Ken Thompson, guided by Rob Pike's design criteria

## 2. Notational conventions

[RFC2119]

UCS characters aer represented as U+HHHH, where HHHH is a string of 4 to 6 hex digits

## 3. UTF-8 definition

In UTF-8, characters from the U+0000..=U+10FFFF range are encoded by using 1 to 4 octets depending on the character

        Range         |            UTF-8 sequence
0000 0000 - 0000 007F | 0xxxxxxx
0000 0080 - 0000 07FF | 110xxxxx 10xxxxxx
0000 0800 - 0000 FFFF | 1110xxxx 10xxxxxx 10xxxxxx
0001 0000 - 0010 FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx

An algorithm to encode a character to UTF-8:

1. Determine the length of the UTF-8 sequence by consulting the table above. There's only one valid length for any symbol

2. Set the bits of the octets as specified by the second column of the table

3. Fill the bits marked by x from the bits of the character number, expressed in binary. The lowest-order bit goes to the lowest-order x position

UTF-8 prohibits encoding character numbers between U+D800 and U+DFFF, which are reserved for the sake of UTF-16 and do not directly represent characters. When converting from UTF-16 to UTF-8, it is necessary to first decode the UTF-16 data to obtaincharacter numbers, and then encode that into UTF-8

AN algorithm to decode a character from UTF-8:

1. Initialize a binary number with at least 21 bits to zero

2. Determine which bits encode the character by consulting the second column of the table above

3. Distribute the x bits from the sequence to the binary number

Implementations MUST protect against decoding invalid sequences

## 4. Syntax of UTF-8 Byte Sequences

```
UTF8-octets             ::= UTF8-char*
UTF8-char               ::= UTF8-1
                          | UTF8-2
                          | UTF8-3
                          | UTF8-4
UTF8-1                  ::= (0x00..=0x7F)
UTF8-2                  ::= (0xC2..=0xDF) UTF8-tail
UTF8-3                  ::= 0xE0 (0xA0..=0xBF) UTF8-tail
                          | (0xE1..=0xEC)      UTF8-tail UTF8-tail
                          | 0xED (0x80..=0x9F) UTF8-tail
                          | (0xEE..=0xEF)      UTF8-tail UTF8-tail
UTF8-4                  ::= 0xF0 (0x90..=0xBF) UTF8-tail UTF8-tail
                          | (0xF1..=0xF3)      UTF8-tail UTF8-tail UTF8-tail
                          | 0xF4 (0x80..=0x8F) UTF8-tail UTF8-tail
UTF8-tail               ::= (0x80..=0xBF)
```

## 5. Versions of the standards

Nothing ever happens

## 6. Byte order mark (BOM)

The UCS character U+FEFF "ZERO WIDTH NO-BREAK SPACE", also known as "BYTE ORDER MARK", apart from its intended usage, can also be prepended to a stream to signify that the stream consists of UCS characters.

U+FEFF MUST NOT be interpreted as a "signature", unless it is at the beginning of the stream. If it is a signature, it may be stripped to prevent weird stuff, for example, when concatinating strings, but it is RECOMMENDED to not do so, unless you have a good reason

U+FEFF in the first position of a stream MAY be interpreted as a zero-width non-breaking space. To completely disambiguate the matter, there is U+2060 "WORD JOINER" with exact same semantics, but without the "signature" functionality. 

Protocols MAY restrict usage of U+FEFF as a signature to remove this uncertainty

- A protocol SHOULD forbid use of U+FEFF as a signature, if all the textual elements are mandated to be UTF-8

- A protocol SHOULD forbid this if it provides some other means for encoding identification

- A protocol SHOULD NOT forbid the use of U+FEFF as a signature is there are no mechanisms to identify the encoding, or when it won't always be in a position to do so.

If a protocol forbids U+FEFF's usage as a signature, it MUST always be interpreted as "ZERO WIDTH NO-BREAK SPACE". If it doesn't forbid, it SHOULD be prepared to handle a signature

## 7. Examples

View the original

## 8. MIME registration

Label "UTF-8" is used to identify this encoding. No versioning is needed, since breaking changes are to never be introduced

## 9. IANA Considerations

Idk

## 10. Security Considerations

Implementers need to consider the security aspects of how to handle illegal sequences.

Pretty much be careful with all the ranges specified by this document and you'll be fine

## 11. Acknowledgements

Some cool people

## 12. Changes from RFC 2279

Read the original
