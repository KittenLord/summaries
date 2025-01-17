Source: https://www.rfc-editor.org/rfc/pdfrfc/rfc3986.txt.pdf

# Uniform Resource Identifier (URI): Generic Syntax

## Abstract

A Uniform Resource Identifier (URI) is a compact path to some resource. Here will be explained URI's syntax, process for resolving relative URIs, also smth about security. 

## 1. Introduction

A URI is a simple way to identify a resource. An older version of URIs is described in [RFC1630]. Syntax is based on [RFC1736] and [RFC1737]

### 1.1 Overview of URIs

URIs are:

Uniform - any type of resource can be accessed with any kind of mechanism using the same uniform syntax

Resource - by "resource" you can imagine pretty much anything (not even necessarily on the Internet)

Identifier - self-explanatory

URIs are not required to be permanently attached to only one resource (though that would be kinda nice)

URIs have a global scope, but may be interpreted differently depending on user (think of localhost - same concept, different for each user)

#### 1.1.1. Generic syntax

A URI begins with a scheme name, which may add extra constraints (basically this document defines a superset of URIs, and different schemes may limit that set)

#### 1.1.2. Examples

Here are some URI examples:

```
ftp://ftp.is.co.za/rfc/rfc1808.txt

http://www.ietf.org/rfc/rfc2396.txt

ldap://[2001:db8::7]/c=GB?objectClass?one

mailto:John.Doe@example.com

news:comp.infosystems.www.servers.unix

tel:+1-816-555-1212

telnet://192.0.2.16:80/

urn:oasis:names:specification:docbook:dtd:xml:4.1.2
```

#### 1.1.3. URI vs URL vs URN

URI (identifier), URL (locator), URN (name)

URL is a URI, which in addition to identifying a resource, provides a means of locating the resource

URN is a URI, which is expected to be globally and temporary persistent

### 1.2. Design considerations

#### 1.2.1. Transcription

The URI syntax was designed to be easily written through any means, whether it is on paper, on computer, or however the fuck you want

Percent-encoded octets can be used to represent stuff that doesn't align to what I said above

#### 1.2.2. Separating Identification from Interaction

URI only provides identification of the resource, it doesn't imply it actually exists

URI resolution - the process of dereferencing a URI

URI retrieval - dereferencing a URI and getting the resource

URI may change what they are pointing at over time

#### 1.2.3. Hierarchical Identifiers

The URI syntax is organized hierarchically, left most significant, right least significant

Generic syntax uses `':' '/' '?' '#'` to delimit components

Relative reference - obvious, allows for resource trees to be independent on protocol, since relative references stay the same

Relative references are NOT a subset of URIs, they are a different method of referencing URIs

### 1.3. Syntax notation

I'll use whatever I feel like

## 2. Characters

The specification does not mandate any specific encoding, it is left up to the protocol (if not, it is assumed to use whatever encoding the surrounding text uses)

### 2.1. Percent-Encoding

Percent-encoding is used to represent a data octet if that octet's symbol is reserved/forbidden. It is encoded as "%XX", where X is a hex digit (case insensitive, but uppercase by default), thus these 3 characters represent 1 byte.

A percent encoding of a reserved character IS DIFFERENT from the reserved character itself. A percent encoding of a non-important character is equivalent to that character itself

### 2.2. Reserved Characters

Generic delimiters - `':' '/' '?' '#' '[' ']' '@'`

Subcomponent delimiters - `'!' '$' '&' '’' '(' ')' '*' '+' ',' ';' '='` (ignore the second backtick)

// TODO: Figure out what the fuck subcomponent delimiters mean

### 2.3. Unreserved characters

Unreserved characters - `'a-z', 'A-Z', '0-9', '-', '.', '_', '~'`

These all are semantically equivalent to their percent-encodings, though they are expected to be encoded in their ASCII representation rather than percend-encodings

### 2.4. When to Encode or Decode

Just decode the components first, and then the percent encodings, so to not fuck everything up (or however you want really, it's not mandatory)

% is always encoded as "%25", and you should be careful to not encode and decode it multiple times and end up with a ton of extra garbage "%25"s

### 2.5. Identifying Data

I don't get why this section exists - pretty much be careful with encodings, and assume that utf-8 is the global default that any sane person should use

## 3. Syntax components

I will not be summarizing most of the prose here, because if you can read the grammar it just won't be needed, and if you can't - that's on you

```
reserved                ::= generic-delims | sub-delims

// Used by URI
generic-delims          ::= ":" | "/" | "?" | "#" | "[" | "]" | "@"

// Potentially used by schemes
sub-delims              ::= "!" | "$" | "&" | "'" | "(" | ")" | "*" | "+" | "," | ";" | "="

unreserved              ::= "a-z" | "A-Z" | "0-9" | "-" | "." | "_" | "~"

// case-insensitive, uppercase by default
percent-encoded         ::= "%" HEXDIGIT HEXDIGIT



URI                     ::= scheme ":" hierarchy-part [ "?" query ] [ "#" fragment ]

// case-insensitive, lowercase by default
scheme                  ::= ALPHA (ALPHA | DIGIT | "+" | "-" | ".")*

hierarchy-part          ::= "//" authority [ path-abempty ]
                          | path-absolute
                          | path-rootless
                          | e

path-absolute           ::= "/" [ segment-nonzero path-abempty ]

path-abempty            ::= ("/" segment)*

path-rootless           ::= segment-nonzero path-abempty

path-noscheme           ::= segment-nonzero-nc path-abempty

segment                 ::= pchar*
segment-nonzero         ::= pchar+
segment-nonzero-nc      ::= pchar-nc+

pchar                   ::= unreserved | percent-encoded | sub-delims | "@" | ":"
pchar-nc                ::= unreserved | percent-encoded | sub-delims | "@"

query                   ::= (pchar | "/" | "?")*

fragment                ::= (pchar | "/" | "?")*



authority               ::= [ userinfo "@" ] host [ ":" port ]

userinfo                ::= (unreserved | percent-encoded | sub-delims | ":" )*

// case-insensitive, lowercase by default
host                    ::= IP-literal
                          | IPv4address
                          | reg-name

IP-literal              ::= "[" ( IPv6address | IPvFuture ) "]"

IPvFuture               ::= [ "v" | "V" ] HEXDIGIT+ "." (unreserved | sub-delims | ":")+

IPv6address             ::= // view section 3.2.2.

IPv4address             ::= dec-octet "." dec-octet "." dec-octet "." dec-octet
dec-octet               ::= // decimal 0..=255

reg-name                ::= (unreserved | percent-encoded | sub-delims)*

port                    ::= DIGIT*
```

### 3.1. Scheme

Scheme is case-insensitive, but lowercase by default

To register a new URI scheme see [BCP35], whatever that is, some additional info [RFC2718]. 

Scheme-specific URI syntax must be a subset of general URI syntax

If URI violates scheme-specific rules it should be discarded, not implicitly "fixed"

### 3.2. Authority

If port is omitted, ":" should also be (that's obvious?)

If authority is specified, the following path may be empty, or begin with a "/" (obvious as well, but important to consider I guess)

#### 3.2.1. User information

Format "user:password" is deprecated, anything following the first ":" in the userinfo component should not be rendered as plaintext, unless it is empty (lol)

In general, userinfo should be very visually distinct whenever rendered

#### 3.2.2. Host

Host is case-insensitive, but lowercase by default

Presence of a host component does not imply that Internet will be used

Being a "host" is not the only purpose of the host field, but it is the most common (???)

The syntax rule is kinda ambigious between IPv4address and reg-name, so IPv4address takes the priority if it can be matched

IP-literal rule is the only place where "[" and "]" are allowed in the URI

IPvFuture is designed to be used in case more IP versions emerge. The "v" part does not indicate the IP version, but the version of the format. If a format specified by "v" is not supported, an "address mechanism not supported" error should be reported.

I'm not gonna bother with this syntax rule, here is how it was laid out originally:

```
IPv6address = 6( h16 ":" ) ls32
            / "::" 5( h16 ":" ) ls32
            / [ h16 ] "::" 4( h16 ":" ) ls32
            / [ *1( h16 ":" ) h16 ] "::" 3( h16 ":" ) ls32
            / [ *2( h16 ":" ) h16 ] "::" 2( h16 ":" ) ls32
            / [ *3( h16 ":" ) h16 ] "::" h16 ":" ls32
            / [ *4( h16 ":" ) h16 ] "::" ls32
            / [ *5( h16 ":" ) h16 ] "::" h16
            / [ *6( h16 ":" ) h16 ] "::"

ls32        = ( h16 ":" h16 ) / IPv4address
; least-significant 32 bits of address

h16         = 1*4HEXDIG
; 16 bits of address represented in hexadecimal

```

// TODO: there's some extra stuff I'll summarize later, but you probably should just read the original section, it is one of the only ones where the verbosity may be needed

#### 3.2.3. Port

Port

### 3.3. Path

If there's an authority, path is either empty or starts with "/". Otherwise, path cannot start with "//" (obvious)

If a path is a relative reference (path-noscheme), first segment cannot have a colon

Path segments "." and ".." are defined for relative references, and are used at the very beginning of a relative reference. They are removed as part of the resolution process

// TODO: reread this one

Useful reference
https://stackoverflow.com/questions/20569958/url-format-empty-path#comment65084610_20569958

### 3.4. Query

Queries may have "/" and "?" represent data, because their semantics are not ambiguous here, since query is everything after the first "?". Some bad implementations may break because of this, but generally it is okay to use them, especially since a query can hold another URI, so you shouldn't percent-encode that stuff

// NOTE: it seems that ?a=b&c=d format is not defined by URI

### 3.5. Fragment

Fragment identifies a secondary resource in relation to the primary resource

Fragment's format and resolution is dependent on the media type [RFC2046]

Fragment semantics are independent of the URI scheme and cannot by redefined by it

Fragment should not be used in the schemes-specific processing of a URI - it is separated, and the user dereferences the fragment with regard to reptrieved resource/representation

Same thing as in section 3.4. with regards to "/" and "?" applies

## 4. Usage

Some reference syntax will be defined here

### 4.1. URI Reference

```
URI-reference       ::= URI
                      | relative-ref

relative-ref        ::= relative-part [ "?" query ] [ "#" fragment ]

relative-part       ::= "//" authority path-abempty
                      | path-absolute
                      | path-noscheme
                      | e

absolute-URI        ::= scheme ":" hierarchy-part [ "?" query ]
```

The URI-reference rule prioritizes the URI production

### 4.2. Relative Reference

The first production of the relative-part rule is called network-path reference

The second - absolute-path reference

The third - relative-path reference

Relative-path reference cannot have a ":" in its first segment due to being ambiguous, use an extra "." segment to fix ("bad:relref" -> "./good:relref")

### 4.3. Absolute URI

I'm not sure why this section exists, it pretty much reiterates that a scheme MUST NOT redefine semantics of the fragment component, and that any URI that scheme-custom syntax for URI rule should also match the absolute-URI rule written above

### 4.4. Same-Document Reference

Either an empty reference, or only has the fragment component - the target of such reference is defined to be within the same entity, so no retrieval should take place

### 4.5. Suffix Reference

Normal URI: `https://www.mywebsite.com/whatever`

Suffix reference: `www.mywebsite.com/whatever`

Suffix references should be avoided whenever possible cuz it's ambiguous

A sender of a representation with relative references is responsible for ensuring that a base URI can be established

## 5. Reference Resolution

### 5.1. Establishing a Base URI

A base URI must conform to the absolute-URI rule

A base URI can be retrieved in ways defined by the following 4 sections, in order from the highest priority to the lowest priority

#### 5.1.1. Base URI Embedded in Content

Some media types may embed a base URI, use it if you can

#### 5.1.2. Base URI from the Encapsulating Entity

If a document is enclosed within another entity, use the base URI of that entity, if there is one

View the original section of you care

#### 5.1.3. Base URI from the Retrieval URI

If a URI was used to retrieve the representation, that URI is the base URI. If redirected requests were involved, the very last URI is the base URI

#### 5.1.4. Default Base URI

Use application context-specific base URI as the base URI

### 5.2. Relative Resolution

Now that you have the base URI, here's the algorithm to resolve it

#### 5.2.1. Pre-parse the Base URI

A component is undefined if its delimiter does not appear in the URI reference. Path is never undefined, but may be empty

Normalization is optional

#### 5.2.2. Transform References

```hs
let (ref.scheme, ref.authority, ref.path, ref.query, ref.fragment) = parse uriRef

target.fragment = ref.fragment

if not strict && ref.scheme == base.scheme then
    undefine ref.scheme

if defined ref.scheme then
    target.scheme = ref.scheme
    target.authority = ref.authority
    target.path = removeDotSegments ref.path
    target.query = ref.query
else if defined ref.authority then
    target.authority = ref.authority
    target.path = removeDotSegments ref.path
    target.query = ref.query
    target.scheme = base.scheme
else if ref.path == "" then
    target.path = base.path
    target.authority = base.authority
    target.scheme = base.scheme
    if defined ref.query then
        target.query = ref.query
    else
        target.query = base.query
else if startsWith "/" ref.path then
    target.path = removeDotSegments ref.path
    target.query = ref.query
    target.authority = base.authority
    target.scheme = base.scheme
else
    target.path = removeDotSegments $ mergePaths base.path ref.path
    target.query = ref.query
    target.authority = base.authority
    target.scheme = base.scheme
```

#### 5.2.3. Merge paths

mergePaths base ref
    | defined base.authority && base.path == "" = ("/" ++ ref.path)
    | otherwise = -- append ref.path to all segments of the base.path, except for the last one

Thank you RFC for not providing a single example!

#### 5.2.4. Remove Dot Segments

If the input begins with "../" or "./", remove that prefix

else if the input begins with "/./" or is equal to "/.", replace the prefix with "/"

else if the input begins with "/../" or is equal to "/..", replace it with "/" and remove the last segment and its preceding "/" (if any) from the output

else if the input is equal to "." or "..", remove that from the input

else move the first path segment from the input to the end of the output, including the initial "/" (if any), up to but not including the next "/"

This could alternatively be implemented with a stack, but you get how it works I hope

### 5.3. Componennt Recomposition

```hs
let result = ""

if defined scheme then
    result = result ++ scheme ++ ":"

if defined authority then
    result = result ++ "//" ++ authority

result = result ++ path

if defined query then
    result = result ++ "?" ++ query

if defined fragment then
    result = result ++ "#" ++ fragment

result
```

A distinction is made between an undefined component and an empty one!

### 5.4. Examples

```
For a base URI: "http://a/b/c/d;p?q"

we get following resolutions:

"g:h"      ->  "g:h"
"g"        ->  "http://a/b/c/g"
"./g"      ->  "http://a/b/c/g"
"g/"       ->  "http://a/b/c/g/"
"/g"       ->  "http://a/g"
"//g"      ->  "http://g"
"?y"       ->  "http://a/b/c/d;p?y"
"g?y"      ->  "http://a/b/c/g?y"
"#s"       ->  "http://a/b/c/d;p?q#s"
"g#s"      ->  "http://a/b/c/g#s"
"g?y#s"    ->  "http://a/b/c/g?y#s"
";x"       ->  "http://a/b/c/;x"
"g;x"      ->  "http://a/b/c/g;x"
"g;x?y#s"  ->  "http://a/b/c/g;x?y#s"
""         ->  "http://a/b/c/d;p?q"
"."        ->  "http://a/b/c/"
"./"       ->  "http://a/b/c/"
".."       ->  "http://a/b/"
"../"      ->  "http://a/b/"
"../g"     ->  "http://a/b/g"
"../.."    ->  "http://a/"
"../../"   ->  "http://a/"
"../../g"  ->  "http://a/g"
```

#### 5.4.2. Abnormal Examples

".." syntax cannot be used to change the authority component

In general, be careful with ".."

```
"../../../g"     ->  "http://a/g"
"../../../../g"  ->  "http://a/g"
```

Also a distinction between a segment that is "." or "..", and a segment that *contains* "." or ".." should be made

```
"/./g"   ->  "http://a/g"
"/../g"  ->  "http://a/g"
"g."     ->  "http://a/b/c/g."
".g"     ->  "http://a/b/c/.g"
"g.."    ->  "http://a/b/c/g.."
"..g"    ->  "http://a/b/c/..g"
```

Some more examples

```
"./../g"      ->  "http://a/b/g"
"./g/."       ->  "http://a/b/c/g/"
"g/./h"       ->  "http://a/b/c/g/h"
"g/../h"      ->  "http://a/b/c/h"
"g;x=1/./y"   ->  "http://a/b/c/g;x=1/y"
"g;x=1/../y"  ->  "http://a/b/c/y"
```

Again, be careful with "/" and "?" with a query or a fragment

```
"g?y/./x"   ->  "http://a/b/c/g?y/./x"
"g?y/../x"  ->  "http://a/b/c/g?y/../x"
"g#s/./x"   ->  "http://a/b/c/g#s/./x"
"g#s/../x"  ->  "http://a/b/c/g#s/../x"
```

Older implementations allow for the scheme in the relative URI, but you shouldn't do this (unless maybe backwards compatibility)

```
"http:g"  ->  strict - "http:g"
              compat - "http://a/b/c/g"
```

## 6. Normalization and Comparison

Its obvious that URI comparison is a useful operation, and that there should be a way to normalize different URIs, which are semantically equivalent

### 6.1. Equivalence

Since we can't directly compare resources, we will be just comparing the URIs

False negatives fundamentally cannot be 100% deduced due to the abstract nature of a resource, but false positives can and must be prevented

Relative references must be first converted to URIs before comparing

If you are comparing URIs to decide whether you need to retrieve the resource you can remove the fragment component

### 6.2. Comparison Ladder

Here are some methods of comparison. They start with the cheapest, but with more false negatives, and end with the most expensive, but with less false positives

#### 6.2.1. Simple String comparison

You should be comparing character-to-character, not byte-to-byte, due to possible encoding stuff (or you can just convert both to utf-8 or smth lol)

Obviously this won't account for any aliases, but this can be prevented if you only use normalized URIs for comparison in advance

#### 6.2.2. Syntax-Based Normalization

Pretty much any technique that considers at least something that was mentioned above

##### 6.2.2.1. Case Normalization

Stuff that is case-insensitive (scheme, host) should be lowercased before comparing (or uppercased for percent-encodings)

##### 6.2.2.2. Percent-Encoding Normalization

Percent-encoded characters that don't need to be percent-encoded should be unpercent-unencoded

##### 6.2.2.3. Path Segment Normalization

Remove the dot segments according to the rules from section 5.2.4.

#### 6.2.3. Scheme-Based Normalization

Well, use whatever optimizations are provided by a specific scheme.

For example, http scheme allows for following URIs to be equivalent:

```
http://example.com          
http://example.com/         -- http defines the empty path as "/"
http://example.com:/        -- port production is empty, so http uses the default port, just as in the 2 examples above
http://example.com:80/      -- http defines the default port to be 80
```

The second example from the ones above is the normal form for the http scheme

If the scheme allows for the empty authority, it should be empty, unless userinfo or port are provided - then the default value should be used (again, if there is one)

Normalization should not remove delimiters if their component is empty, unless the scheme allows to do so. "http://example.com/?" cannot be assumed to be equivalent to the examples above, even though the query component is empty

The fragment component is not subject to any scheme-based normalization

Other scheme-specific normalizations are possible

### 6.2.4. Protocol-Based Normalization

I'm not sure what is this section about, but summarizing - if you observe that the representations retrieved are the same, and the URIs only differ in the final "/", you may want to consider them equivalent, if it makes sense in the context of the scheme

## 7. Security Considerations

### 7.1. Reliability and Consistency

There is NO guarantee that retrieving a resource identified by the same URI will result in the same representation

A specific scheme may define additional semantics to address this

### 7.2. Malicious Construction

An application should be careful with dereferencing a URI with an unfamiliar port specified, cuz it might inject a bad action where it was not expected by the user (especially ports 0-1023)

Stuff like percent-encoded CR and LF characters should also be treated with care, cuz they might function as delimiters and fuck everything up

### 7.3. Back-End Transcoding

Again, percent-encoded stuff should be carefully taken care of, especially if it is %00, cuz I hope it's obvious how that can fuck shit up

Also if the server needs to access a file to dereference the URI, you need to be careful with that for obvious reasons

### 7.4. Rare IP Address Formats

Be careful to only parse IP Addresses as specified in this document!

### 7.5. Sensitive Information

Secret username and password should not be provided (especially in plaintext). A password within the userinfo component should either be considered an error or ignored, unless for some reason it's intended to be public

### 7.6. Semantic Attacks

The userinfo component could be used to look like something that it is not (impostor amogus). Thus, userinfo should be clearly visually distinct, so users don't get fooled

## 8. IANA Considerations

I don't know what that means

## 9. Acknowledgements

Some cool people

## 10. References

Go view the original

## Appendix A. Collected ABNF for URI

I have already provided everything above

## Appendix B. Parsing a URI Reference with a Regular Expression

```
A regex to parse a URI

    ^(([^:/?#]+):)?(//([^/?#]*))?([^?#]*)(\?([^#]*))?(#(.*))?
     12            3  4          5       6  7        8 9

parses the following URI:

    "http://www.ics.uci.edu/pub/ietf/uri/#Related"

info this:

    $1 = http:
    $2 = http
    $3 = //www.ics.uci.edu
    $4 = www.ics.uci.edu
    $5 = /pub/ietf/uri/
    $6 = <undefined>
    $7 = <undefined>
    $8 = #Related
    $9 = Related

where

    scheme = $2
    authority = $4
    path = $5
    query = $7
    fragment = $9
```

## Appendix C. Delimiting a URI in Context

URIs in plaintext are usually delimited by "uri", <uri>, or

    uri

these delimiters are not a pard of the URI

Whitespace can be used to break up a long URI into multiple lines (this whitespace is ignored), but no whitespace should be introduced after a "-"

You may see a prefix "URL:" serving the same purpose, but it is no longer recommended

## Appendix D. Changes from RFC 2396

Read the original if you're interested
