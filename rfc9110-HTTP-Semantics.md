Source: https://www.rfc-editor.org/rfc/rfc9110.html

# RFC 9110: HTTP Semantics

## Abstract

The Hypertext Transfer Protocol (HTTP) is a stateless application-level protocol for hypertext information systems. This document describes the architecture, terminology, and aspects shared by all HTTP versions. Here will also be described the "http" and "https" URI schemes

## 1. Introduction

### 1.1. Purpose

HTTP hides the implementation details of a service behind a uniform interface, that is independent on the types of resources provided. This goes in both ways, so servers are not aware of each client's purpose.

### 1.2. History and Evolution

There exists HTTP/1.1 (introduced in 1995), HTTP/2 (with a brand new multiplexed session layer, whatever that is) and HTTP/3 (uses UDP instead of TCP).

All of these versions rely on the semantics defined by this document. Neither of them are obsolete, since each of them has its benefits.

### 1.3. Core Semantics

Each message is a request or a response. Client forms and sends requests to the server, asking the server to do or retrieve something. The server listens to requests, interprets each accordingly, and responds to that request with 1 or more responses. The client receives the responses and sees what it can do with them.

HTTP semantics include request methods, header fields, response status codes, resource metadata and whatever else.

### 1.4. Specifications Obsoleted by This Document

I don't care

## 2. Conformance

### 2.1. Syntax Notation

I'll be using whatever I'll happen to be using

### 2.2. Requirements Notation

RFC2119, RFC8174

A sender MUST NOT generate protocol elements that do not match the defined syntax. They almost MUST NOT generate elements that do not fit their role as a participant

A recipient of any message MAY employ workarounds for known erroneous implementations

### 2.3. Length Requirements

A recipient SHOULD parse received protocol elements defensively, relying only on the syntax correctness

HTTP does not define length limitations for most elements, so participants should have their own expectations

A recipient MUST be able to parse and process element lengths that are at least as long as the values that it itself generates in other messages, e.g. if a server publishes very long URI references, it MUST be able to process URI references of the same length or longer

### 2.4. Error Handling

A recipient MUST interpret received elements according to this specification, unless it has been determined that the sender itself is incorrectly implemented.

Unless noted otherwise, a recipient MAY try to recover a valid element from an invalid construct. Such recovery mechanisms will not be discussed here

### 2.5. Protocol Version

HTTP version consists of a major version and a minor version, both decimal digits, separated by a ".". Major version indicates the messaging syntax, minor version indicates the highest minor version that the sender is conformant to

HTTP major version number in case of an introduction of a backwards incompatible message syntax. Minor version is only about adding extra semantics

If a minor version is not defined, it is implied to be "0"

## 3. Terminology and Core Concepts

### 3.1. Resources

A resource is pretty much anything that can be retrieved

HTTP relies on URI for resource identification

A resource cannot treat a request in a mannor inconsistent wiht the semantics of the method of the request

### 3.2. Representations

A representation is whatever you get from a resource at any given time.

Blah blah doesn't matter

### 3.3. Connections, Clients and Servers

HTTP is a client/server protocol

An HTTP client is a program that connects to a server to send requests. An HTTP server is a program that accepts connections to serve requests by sending responses

HTTP is defined as a stateless protocol. Regardless if the TCP connection, or anything else, indicates the "sameness" of the connection, the responses MUST [this MUST is my addition] assume no state.

A server MUST NOT assume that two requests on the same connection are from the same user agent, unless the connection is secured.

### 3.4. Messages

A request has a method and target, it also may contain header fields, client information, representation metadata, content, trailer fields.

A response has a status code, header fields, resource metadata, representation metadata, content, trailer fields.

### 3.5. User Agens

"User agent" is a client program

The most intuitive user agent is a web browser, but that's obviously far from the only one. Therefore, a server shouldn't assume that there's a human on the other side.

### 3.6. Origin Server

"Origin server" is a server that generates responses. Most intuitive are public websites, but again, they are far from the only ones.

### 3.7. Intermediaries

HTTP enables the use of intermediaries to deliver requests/responses. There are 3 common forms of HTTP intermediary: proxy, gateway, tunnel.

These may be used to redirect the requests and responses, or, depending on the request, act as an origin server and generate responses themselves, or whatever the fuck you can come up with.

All messages flow from "upstream" to "downstream". "Inbound" means "toward the origin server", "outbound" means "toward the user agent"

A proxy is a message-forwarding agent *chosen by the client* that acts on behalf of the client and acts as a... proxy.

A gateway, also known as reverse proxy, is a proxy, but for the origin server. It behaves as an origin server from outside, but internally it redirects stuff from and to the real origin server. All HTTP requirements to the origin server also apply to the outbound communication of a gateway.

A tunnel is a blind relay between two connections. Once active, it is not considered a party to the HTTP communication. A tunnel ceases to exist when both ends of the connection are closed.

### 3.8. Caches

Any client or server MAY employ a cache, though it cannot be used while acting as a tunnel.

It's obvious what caching is and what its implications are. More is defined in [RFC9111]

### 3.9. Example Message Exchange

Client request:
```
GET /hello.txt HTTP/1.1
User-Agent: curl/7.64.1
Host: www.example.com
Accept-Language: en, mi
```

Server response:
```
HTTP/1.1 200 OK
Date: Mon, 27 Jul 2009 12:28:53 GMT
Server: Apache
Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
ETag: "34aa387-d-1568eb00"
Accept-Ranges: bytes
Content-Length: 51
Vary: Accept-Encoding
Content-Type: text/plain

Hello World! My content includes a trailing CRLF.
```

## 4. Identifiers in HTTP

### 4.1. URI References

Read the RFC 3986 about URIs (this is actually important!!!).

Apart from grammer defined by that RFC, there are 2 more rules:
```
-- Allows for consecutive "/", unlike path-absolute
absolute-path   ::= ("/" segment)+

-- Does not have a fragment component, unlike relative-ref
partial-URI     ::= relative-part [ "?" query ]
```

Unless otherwise stated, URI references are parsed relative to the target URI

It is RECOMMENDED that all senders and recipients support URIs at least 800 octets long

### 4.2. HTTP-Related URI Schemes

Useful link: https://www.iana.org/assignments/uri-schemes/

HTTP makes use of two schemes - "http" and "https"

#### 4.2.1. http URI Scheme

```
http-URI                    ::= "http" ":" "//" authority path-abempty [ "?" query ]
```

Origin server is identified by the authority component, which includes a host identifier and optional port number. The default port value is 80.

A sender MUST NOT generate an "http" URI with an empty host identifier. Such URIs MUST be rejected as invalid.

#### 4.2.2. https URI Scheme

Origin server has to be capable of establishing a TLS connection

```
https-URI                    ::= "https" ":" "//" authority path-abempty [ "?" query ]
```

Everything from the previous section applies, except that the default port is 443

A client MUST ensure that its HTTP requests are secured, prior to being communicated, and only accept responses to those requests.

Resources available via the "https" scheme are completely different from "http" ones. They are completely distinct. However, extensions to HTTP that rely only on the sameness of the host, such as the Cookie protocol, ought to be very careful with leaking information obtained from a secured connection to an unsecured one.

#### 4.2.3. http(s) Normalization and Comparison

http and https URIs are normalized and compared as per URI rfc.

HTTP does not require using a specific method for determining equivalence.

Scheme-based URI normalization for "http" and "https" has some extra rules
    - If the port is equal to the default, the normal form is to omit the port subcomponent
    - Unless it is used as tht target for an OPTIONS request, an empty path component is equivalent to "/", so the normalized for is the latter
    - scheme and host are case-insensitive and are normalized to lowercase, all other components are case-sensitive.
    - Characters other than those in the reserved set are equivalent to their percent-encoded octets, the normal form is to not encode them

If two HTTP URIs are equivalent after normalization, they can be assume to identify the same resource. Any HTTP component MAY perform normalization. Distinct resources SHOULD NOT be identified by HTTP URIs that are equivalent after normalization

#### 4.2.4. Deprecation of userinfo in http(s) URIs

Format of "user:password" in userinfo component is deprecated.

A sender MUST NOT generate the userinfo subcomponent and its "@" delimiter

Before using a received URI reference, a recipient SHOULD parse for userinfo and treat its presence as an error.

#### 4.2.5. http(s) References with Fragment Identifiers

Some protocol elements allow for the fragment component, others don't - this will be indicated (see 4.1.)

### 4.3. Authoritative Access

Authoritative access - dereferencing an identifier in an authoritative way (controlled by the resource owner).

I don't really get what they're trying to say here

#### 4.3.1. URI Origin

The "origin" for a given URI is a triple (scheme, host, port). If URI does not define a port, the default one is used

In the URI representation the port is always present!

```
URI: https://Example.Com/happy.js
Origin: ("https", "example.com", 443)
Origin (URI representation): https://example.com:443
```

Each origin is distinct from any other (even if they only differ by port or smth).

#### 4.3.2. http Origins

Authority is whoever controls the origin server defined by the URI authority component

A client MAY attempt access by resolving the host identifier to an IP address and send HTTP request message to that, the request containing a request target that matches the client's target URI.

If the server responds to such a request with a non-interim HTTP response message, that response is considered an authoritative answer.

#### 4.3.3. https Origins

https scheme associates authority based on server possessing private key corresponding to a certificate trusted by client.

In <=HTTP/1.1 client will only attribute authority to a server when they are communicating over an established secured connection to that URI origin's host.

In HTTP/2 and HTTP/3 the URI origin's host needs to match any of the hosts present in the server's certificate.

The origin server also must confirm that the port in the URI is the same as the port that the server operates on

// TODO: reread this

#### 4.3.4. https Certificate Verification

To establish a connection a client MUST verify that the service's identity is an acceptable match for the URI's origin server.

A client MUST verify the service identity using the verification process in RFC6125 section 6. The client MUST construct a reference identity from the service's host: if the host is an IP address - IP-ID, otherwise DNS-ID

A reference identity of type CN-ID MUST NOT be used by clients.

If the certificate is not valid for the target URI's origin, a user agent MUST either obtain confirmation from the user before proceeding, or terminate the connection with a bad certificate error. Automated clients MUST log the error and SHOULD terminate the connection with the same error. Automated clients MAY provide a configuration setting to disable this check, but MUST provide a setting to enable it.

#### 4.3.5. IP-ID Reference Identity

Use of IP-ID is not defined for IP versions other than IPv4 and IPv6. 

## 5. Fields

A field is a name/value pair to provide data. They are sent and received within the header and trailer sections of messages.

### 5.1. Field Names

Field names are case-insensitive and ought to be registered within HTTP Field Name Registry

The interpretation of a field does not change between minor versions, though the behavior in the absence of such a field can change. Unless stated otherwise, fields are defined for all version of HTTP, especially Host and Connection fields.

New fields can be introduced if they can be safely ignored.

A proxy MUST forward unrecognized header fields unless the field name is listed in the Connection header field, or the proxy is configured to block or transform such fields. Other recepients SHOULD ignore unrecognized header and trailer fields.

### 5.2. Field Lines and Combined Field Value

Field sections are composed of field lines, each with a field name and a field line value.

When a field name occurs only once, it's obvious what the value is. If it repeats, its value consists of the list of corresponding field line values within that section, concatenated in order and joined with a "," (and optional whitespace)

```
Example-Field: Foo, Bar
Example-Field: Baz

// Example-Field value: "Foo, Bar, Baz"
```

### 5.3. Field Order

A recipient MAY combine multiple field lines within a field section into one, by appending each subsequent field line value in order, separated by a comma and optional whitespace

A proxy MUST NOT change the order of these field line values, the order may be significant

A sender MUST NOT generate multiple field lines with the same name, or append a field line with the name that is already present, unless that field's definition allows for this.

Set-Cookie header field is a notable exception - it may be present multiple times, but it doesn't use the list syntax (comma separation), and it is ought to be treated as a special case

The order of field lines with different field names doesn't matter. Though a good practice is to put the control data first

A server MUST NOT apply a request to the target resource until it receives the entire request header section

### 5.4. Field Limits

There are no limits defined by this spec

If the length of a field line, field value or whatever is larger than the server decides to accept, it MUST respond with an appropriate 4xx status code.

A client MAY discard or truncate received field lines that are larger than the client wishes to process, if it can be safely done.

### 5.5. Field Values

```
field-value             ::= field-content*
field-content           ::= field-vchar [ (SP | HTAB | field-vchar)+ field-vchar ]
field-vchar             ::= VCHAR
                          | obs-text
obs-text                ::= // 0x80..0xFF
```

NOTE: if I'm reading the original spec correctly, it implies that field-content cannot contain only 2 field-vchars (either 1 or 1 + [ (n >= 1) + 1 ]). Either I, or spec authors, are retarded, and I humbly assume it's the former

A field value does not include leading or trailing whitespace. A field parsing implementation MUST exclude such whitespace prior to evaluating the field value.

You pretty much should only use VCHAR (US-ASCII octets), SP (' ') and HTAB ('\t'). A recipient SHOULD treat other allowed octets in a field content as opaque data (obs-text)

Field values containing CR, LF or NUL are invalid and dangerous and bad. A recipient of CR, LF or UNL within a field value MUST either reject the message, or replace each of those characters with SP ' ' before further processing or forwarding. Same applies for other control characters, but recipients MAY retain such characters when they appear within a safe context

Fields that only anticipate a single member as value are called "singleton fields", in contrast to "list-based fields"

You should also be careful with commas for obvious reasons - they should be enclosed in delimiters, like "", and this will be further discussed later.

When a field allows for both quoted and unquoted variants, the meaning of a parameter ought to be the same.

### 5.6. Common Rules for Defining Field Values

#### 5.6.1. Lists (#rule ABNF Extension)

I'll be using # instead of * and #+ instead of + to indivate a comma-joined list (and optional whitespace)

##### 5.6.1.1. Sender Requirements

A sender MUST NOT generate empty list elements

##### 5.6.1.2. Recipient Requirements

Empty lsit elements do not contribute to the count of elements. A recipient MUST parse and ignore empty list elements, but may not commit (ignore several empty list elements, that can be attributed to a common mistake, then reject)

Pretty much just expect that some retard may use empty list elements

#### 5.6.2. Tokens

```
token           ::= tchar+
tchar           ::= "!" | '#' | "$" | "%" | "&" | "'" | "*" | "+" | "-" | "." | "^" | "_" | "`" | "|" | DIGIT | ALPHA

// these are specifically not allowed in a token
// delimiters   ::= DQUOTE | "(" | ")" | "," | "/" | ":" | ";" | "<" | "=" | ">" | "?" | "@" | "[" | "]" | "\" | "{" | "}"
```

#### 5.6.3. Whitespace

OWS - optional whitespace (if for readability a sender SHOULD genereate OWS as a singular ' ', otherwise SHOULD NOT)

RWS - required whitespace (a sender SHOULD generate RWS as a single ' ')

BWS - bad whitespace (a sender MUST NOT generate BWS. A sender MUST parse for BWS and remove it (it may be there for historical reasons))

#### 5.6.4. Quoted Strings

A quoted strings is two delimiting "", with any field-vchar inbetween, except for '"' and '\'. The '\' has a special function of quoting the next character, whatever the character is. This combination is called quoted-pair

A sender SHOULD NOT generate a quoted-pair in a quoted-string except to quote '"' or '\'. A sender SHOULD NOT generate a quoted-pair in a comment except where necessary to quote '(' and ')'and backslash octets

#### 5.6.5. Comments

A comment is anything surrounded by "()". Comments can be "nested", '(' and ')' can be escaped if needed. They are only allowed in field values when is is said explicitly

#### 5.6.6. Parameters

Parameters are a common syntax to encode structs within one field value

```
parameters          ::= (OWS ";" OWS [parameter])*
parameter           ::= parameter-name "=" parameter-value
parameter-name      ::= token
parameter-value     ::= token
                      | quoted-string
```

Note a semicolon before *each* parameter, not between, and also lack of OWS around '='

Parameter names are case-insensitive, values might or might not be

Parameter value as token is equivalent if it made a quoted-string

#### 5.6.7. Date/Time Formats

```
HTTP-date           ::= IMF-fixdate | obsolete-date

// Example of preferred format:
// "Sun, 06 Nov 1994 08:49:37 GMT"  - IMF-fixdate

// Examples of two obsolete formats:
// "Sunday, 06-Nov-94 08:49:37 GMT" - RFC 850 format
// "Sun Nov  6 08:49:37 1994"       - ANSI C asctime() format

IMF-fixdate         ::= day-name "," " " date1 " " time-of-day " " "GMT"

day-name            ::= "Mon" | "Tue" | "Wed" | "Thu" | "Fri" | "Sat" | "Sun"

date1               ::= day " " month " " year
day                 ::= DIGIT DIGIT
month               ::= "Jan" | "Feb" | "Mar" | "Apr" | "May" | "Jun" | "Jul" | "Aug" | "Sep" | "Oct" | "Nov" | "Dec"
year                ::= DIGIT DIGIT DIGIT DIGIT

time-of-day         ::= hour ":" minute ":" second
hour                ::= DIGIT DIGIT
minute              ::= DIGIT DIGIT
second              ::= DIGIT DIGIT

// Obsolete formats:
obsolete-date       ::= rfc850-date | asctime-date

rfc850-date         ::= day-name-l "," " " date2 " " time-of-day " " "GMT"
date2               ::= day "-" month "-" DIGIT DIGIT
day-name-l          ::= "Monday" | "Tuesday" | "Wednesday" | "Thursday" | "Friday" | "Saturday" | "Sunday"

asctime-date        ::= day-name " " date3 " " time-of-day " " year
date3               ::= month " " (DIGIT DIGIT | " " DIGIT)
```

A recipient MUST accept all three HTTP-date formats. A sender MUST generate timestamps in the IMF-fixdate format

A HTTP-date represents UTC time. First two format indicate this by "GMT" part, for the third it is implied

A clock used to generate UTC time ought to use NTP (RFC5904) to sync with UTC.

HTTP-date is case sensitive

A sender MUST NOT generate additional whitespace in an HTTP-date

Recipients of rfc850-date MUST interpret the year as the last year that had that number (i.e. as of writing "48" means 1948, in year 2069 "48" will mean 2048)

You can obviously use other timestamp formats internally

## 6. Message Abstraction

Each HTTP versions defines syntax differently, but overall general details and semantics will be defined here

A message consists of the following:
    - control data to describe and route the message
    - a headers lookup table of name/value pairs
    - a potentially unbounded stream of content
    - a trailers lookup table of name/value pairs

Messages are expected to be processed as a stream, so content is being revealed while being read. Control data describes what needs to be known immediately, header fields describe what needs to be known about the content, and trailer fields provide extra metadata

Messages are intended to be "self-descriptive" and "self-contained". A client MUST retain knowledge of the request when parsing the response

### 6.1. Framing and Completeness

Message Framing indicates the boundaries of each message.

HTTP/0.9 and early HTTP/1.0 used the connection closure as the ending boundary. This implicit framing is also allowed in HTTP/1.1, but it is ambiguous with the other participant just crashing, so it's better to use explicit framing.

A message is considered "complete" when all of the octets indicated by its framing are available. This doesn't apply to implicit framing, unless a transport-level can explicitly indicate that it was not complete

### 6.2. Control Data

Request's control data includes a request method, target, and protocol version. Response's control data includes a status code, optional reason phrase, and protocol version

In <=HTTP/1.1 control data is sent as the first line of a message. In HTTP/2 and HTTP/3 it is sent as pseudo-header fields with reserved names.

Every HTTP message has a protocol version. In might be identified explicitly, or be inferred by the connection in use.

When a message is forwarded by an intermediary, the protocol version is updated to intermediary's. The Via header field is used to communicate the upstream protocol information within a forwarded message.

A client SHOULD send the highest version it supports, where major version is no higher than the server's, if it is known. A client MUST NOT send a version it doesn't supprot.

A client MAY send a lower request version if it is know that the server incorrectly implemented HTTP spec, but only after at least one normal request has been attempted.

A server SHOULD send the highest version it supports, that has a major version less than or equal to the request's. A server MUST NOT send a version to which it is not conformant. A server can send a 505 (HTTP Version Not Supported) for any version to refuse the client's major version.

A recipient that receives a message with a major version that it implements and a minor that it doesn't it SHOULD process the message as if it were in the highest minor version it supports. A recipient may assume that such message is sufficiently backwards-compatible.

### 6.3. Header Fields

Fields in the header are "header fields" or "headers".

### 6.4. Content

I'm not sure what they're saying here

#### 6.4.1. Content Semantics

The purpose of content in a request is defined by the method semantics (for example, a representation in the content of a PUT request represents the desired state of the target after the request is applied, while in POST it represents information to be processed by the target)

In a response, content's purpose is defined by the request method, response status code and response fields. The content of a 200 response to GET represents the current state of the target resource, while 200 to POST might represent the processing result of the new state

Code 206 to GET indicates a 1+ parts of a representation

Responses to the HEAD method never include content

2xx responses to a CONNECT method switch the connection to tunnel mode instead of having content

1xx, 204 and 304 responses do not include content

All other responses include content, possibly with zero length

#### 6.4.2. Identifying Content

Sometimes it's useful to attach the identifier for the resource you're sending

For a request:
    - If the request has a Content-Location header field, the sender asserts that the content is a representation of the resource at that identifier, however, such assertion cannot be trusted (no means to determine truthness defined by this spec specifically)
    - Otherwise, the content is unidentified by HTTP, but the content itself might provide some info

For a response:
    1. If the request is HEAD or response is 204 or 304, there is no content
    2. If the request is GET and response is 200, the content is a representation of the target resource
    3. If the request is GET and response is 203 - modified or enhanced representation of the target resource
    4. If the request is GET and response is 206 - a part of target resource
    5. If the response has a Content-Location and it matches the target URI, that is the target resource
    6. If the response has a Content-Location header field and it differs from the target URI, Content-Location value takes priority. However, this again cannot be trusted.
    7. Otherwise, the content is unidentified by HTTP

### 6.5. Trailer Fields

Fields in the trailer section are "trailer fields" or "trailers". They may supply integrity checks, signatures, metrics or post-processing status information

Trailer fields ought to be processed and stored separately from the header fields to avoid contradicting message semantics known at the time the header was complete.

#### 6.5.1. Limitations on User of Trailers

A trailer section is only possibly when supported by used HTTP version and enabled by explicit framing.

A sender MUST NOT generate a trailer field unless the sender knows the corresponding header field name's definition parmits the field to be sent in trailers

A recipient MUST NOT merge a trailer field into a header section unless the recipients understands the corresponding header field definition and that definition explicitly permits and defines how trailer field values can be safely merged.

The presence of the keyword "trailers" in the TE header field of a request indicates that the client and all downstream clients are willing to accept trailer fields (this doesn't mean that they will process them, only not drop them)

A server SHOULD NOT generate trailer fields that it believes are necessary for the user agent to receive

#### 6.5.2. Processing Trailer Fields

The Trailer header field may indicate fields likely to be sent in the trailer section.

Trailer fields that might be generated more than once MUST be defined as a list-based field

At the end of a message, a recipient MAY treat the set of received trailer fields as a data structure of name/value pairs, similar to the header fields.

### 6.6. Message Metadata

#### 6.6.1. Date

Date header field represents the date and time at which the message was originated.

```
Date ::= HTTP-date
```

A sender SHOULD generate the Date value as the best available approximation of the date and time of message generation (the moment just before generating the message content) .

An origin server with a clock MUST generate a Date header in all 2xx 3xx and 4xx responses, and MAY generate a Date header in 1xx and 5xx responses.

An origin server without a clock MUST NOT generate a Date header field

A recipient with a clock that receives a response without a Date header MUST record the time it was received and append that Date header field if it is cached or forwarded downstream. If the Date is invalid, a recipient MAY replace that value in a similar fashion

A user agent MAY send a Date header field in a request, though usually it's not necessary.

#### 6.6.2. Trailer

Trailer header field provides a list of field names that the sender anticipates sending as trailer fields with that message.

```
Trailer ::= field-name#
```

If a sender intends to generate trailer fields in SHOULD generate a Trailer header field.

If an intermediary discards the trailer section, the Trailer field could be a hint to what was lost.

## 7. Routing HTTP Messages

Routing is determined by each client based on the target resource, proxy configuration, establishment or reuse of an inbound connection. The response routing follows the same connection chain back to the client

### 7.1. Determining the Target Resource

A URI reference is resolved to its absolute form in order to obtain the target URI. The target URI excludes the fragment component.

A client needs to send enough URI components to identify the target resource.

The target URI components, referred to as "request target", are sent within the message control data and the Host header field

There are two exceptions:
    - For CONNECT the request target is the host name and port number, seaprated by a ":"
    - For OPTIONS the request target can be simply "*"

Above two forms MUST NOT be used with other methods

Upon receipt of a client's request, a server reconstructrs the target URI from the received components. This process is specific to each major version. In the past, recomposed target URI have been distinctly referred to as "effective request URI"

### 7.2. Host and :authority

```
// uri-host ::= host defined in URI RFC
Host ::= uri-host [ ":" port ]
```

In HTTP/2 and HTTP/3 Host header is sometimes replaced by the ":authority" pseudo-header

A user agent MUST generate a Host header, unless an ":authority" pseudo-header is used instead. A user agent that does send it SHOULD send it as the first field in the header section

You should be careful when using Host for rerouting or whatever

### 7.3. Routing Inbound Requests

A client may decide to direct the request

#### 7.3.1. To a Cache

If it can be cached, it is usually directed to cache first

#### 7.3.2. To a Proxy

If a client is configured to use proxy, it directs to the proxy

#### 7.3.3. To the Origin

Otherwise, directly to the origin

// TODO: reread sections 4.3.2. and 4.3.3., they seem to be relatively important, albeit fucking incomprehensible

### 7.4. Rejecting Misdirected Requests

Once a request is received by a server and parsed sufficiently to determine the target URI, the server decides to process the request, forward it, redirect the client to a different source, respond with an error, or drop the connection.

For example, if server match the information passed in the Host header, the server might decide to drop the connection, unless it comes from a trusted gateway as a simple mistake

Unless the connection is from a trusted gateway, an origin server MUST reject a request if any scheme-specific requirements for the target URI are not met. A request for an "https" resource MUST be rejected unless it has been received over a connection that has been secured via a certificate valid for that target URI's origin

The 421 status code in a response indicates that the server deemed the request as misdirected

### 7.5. Response Correlation

All responses, regardless of the status code, can be send at any time after a request is receievd, even if the request is not yet complete. Clients are not expected to wait any specific amount of time for a response, but may drop the connection after a reasonable amount of time without a response

### 7.6. Message Forwarding

Intermediaries are expected to forward messages even when protocol elements are not recognized

An intermediary not acting as a tunnel MUST implement the connection header field, and exclude fields from being forwarded that are only intended for the incoming connection

An intermediary MUST NOT forward a message to itself, unless it can somehow prevent an infinite loop

#### 7.6.1. Connection

Connection header specifies control options desired by the sender for the current connection. This is case-insensitive

```
Connection          ::= connection-option#
connection-option   ::= token
```

When a  field that's not Connection supplies control information for a connection, the sender MUST list that field name within the Connection header field. Some version of HTTP do not allow the Connection field

Intermediaries MUST parse a received Connection header field before forwarding, and remove any header/trailer field listed in Connection, including itself (or replace the Connection with something of its own)

Intermediaries SHOULD remove or replace fields that are known to require removal regardless. Some of these are:

- Proxy-Connection
- Keep-Alive
- TE
- Transfer-Encoding
- Upgrade

A sender MUST NOT send a connection option that is indented for all recipients of the content.

Connection options may not be actual fields, since they are just tokens, and may not need a value. When defining such an option, you should still reserve the corresponding field name.

#### 7.6.2. Max-Forwards

A header for TRACE and OPTIONS request methods to limit the number of forwards by proxies. The value indicates remaining forwards

```
Max-Forwards ::= DIGIT+
```

Upon receiving a TRACE or OPTIONS request with this header, an intermediary MUST check and update its value before forwarding. If they value is 0, the intermediary MUST NOT forward the request, and instead it MUST respond as the final recipient. If it's greater than zero, the new value is max(Max-Forwards - 1, maxSupportedValueByTheRecipient)

If the request isn't TRACE nor OPTIONS, a recipient MAY ignore it.

#### 7.6.3. Via

Via is used to indicate intermediate protocols and recipients.

```
Via                     ::= (received-protocol RWS received-by [ RWS comment ])#
received-protocol       ::= [ protocol-name "/" ] protocol-version
received-by             ::= pseudonym [ ":" port ]
pseudonym               ::= token
```

Each member represents a proxy or gateway that has forwarded the message. Each intermediary appends it own information

A proxy MUST send an appropriate Via header field in each message it forwards. An HTTP-to-HTTP gateway MUST send an appropriate Via header field in each inbound request message and MAY send a Via header field in response messages.

Protocol-name is omitted when the received protocol is HTTP

The received-by portion is normally the host and port, however, a sender MAY replace it with a pseudonym for any reason. If a port is not provided, the default port MAY be assumed, if received-protocol specifies one.

A sender MAY generate comments, a recipient MAY remove them.

An intermediary used as a portal through a network firewall SHOULD NOT forward the names and ports of hosts within the firewall region, unless explicitly enabled to do so. It SHOULD replace them by appropriate pseudonyms otherwise.

An intermediary MAY combine contiguous members of a list in a single member, if the protocol and version are the same, for example

```
Via: 1.0 aaa, 1.1 bbb, 1.1 ccc, 1.0 ddd
->
Via: 1.0 aaa, 1.1 aux, 1.0 ddd
```

A sender SHOULD NOT combine list members unless they are under the same organizational control and use pseudonyms. A sender MUST NOT combine members with different protocols.

### 7.7. Message Transformations

An HTTP-to-HTTP proxy is called a "transforming proxy" if it modifies messages in a semantically meaningful way.

If a proxy receives a target URI with a host name that is not a fully qualified domain name, it MAY add its own domain to the hest name it received when forwarding the request. If it is fully qualified, a proxy MUST NOT change it.

A proxy MUST NOT modify the "absolute-path" and "query" of the targer URI, except as required by protocol (for example "" -> "/")

A proxy MUST NOT transform the content of a response that contains a no-transform cache directive. Otherwise, it MAY transform it, this transformation may be indicating by changing 200 response code to 203

A proxy SHOULD NOT modify header fields that provide information about the endpoints of the communication chain, the resource state, or the selected representation unless the field's definition allows such modification, or it is deemed necessary for privacy or security

### 7.8. Upgrade

Upgrade is used to transition from HTTP/1.1 to another protocol on the same connection

A client MAY send a list of protocol names in order of descending preference. A server MAY ignore a received Upgrade header.

```
Upgrade             ::= protocol#
protocol            ::= protocol-name [ "/" protocol-version ]
protocol-name       ::= token
protocol-version    ::= token
```

Recipients SHOULD use case-insensitive comparison for matching protocol-name

A server that sends a 101 response MUST send an Upgrade header to indicate the new protocol(s). If multiple protocol layers are being switched, the sender MUST list the protocols in layer-ascending order. A server MUST NOT switch to a protocol that wasn't indicated by the client. A server MAY choose to ignore the order of preference.

A server that sends a 426 response MUST send an Upgrade header, descending preference

A server MAY send an Upgrade header in any other response to show available options, descending preference

Immediately after sending a 101 response, the server is expected to continue responding in a new protocol, even to that same request.

A server MUST NOT switch protocols unless the received message semantics can be honored by the new protocol (since the change occurs immediately to the same request). An OPTIONS request can be honored by any protocol.

A sender of Upgrade MUST also send an "Upgrade" connection option in the Connection header field. A server receiving a HTTP/1.0 request MUST ignore the Upgrade header field.

A client can only begin using an upgraded protocol after finishing sending the request. If a server receives both an Upgrade and an Expect "100-continue" headers, the server MUST send a 100 response before sending a 101 response

The Upgrade header obviously cannot change the transport protocol, nor change the connection. For that one may use 3xx responses.

This spec only defines the protocol name "HTTP" - other protocol names ought to be registered separately.

## 8. Representation Data and Metadata

### 8.1. Representation Data

The representation data is in format and encoding defined by the header fields, namely, Content-Type and Content-Encoding

```
representation-data := Content-Encoding $ Content-Type data
```

### 8.2. Representation Metadata

In a response to a HEAD request, the representation header fields describe the representation data that would've been responded to a GET request

### 8.3. Content-Type

Content-Type indicates the media type of the associated representation. This defines both the data format and how to process it.

A sender that generates a message with content SHOULD generate a Content-Type header, unless the type is unknown to the sender. If Content-Type is not present, recipient MAY assume "application/octet-stream", or somehow determine the type.

You should be careful when trying to decide the data type for yourself (especially when sniffing, and user should be able to disable sniffing, whatever that is)

```
Content-Type ::= media-type
```

#### 8.3.1. Media Type

HTTP uses media types defined in RFC2046 (these are case-insensitive, but lowercase preferred)

```
media-type  ::= type "/" subtype parameters
type        ::= token
subtype     ::= token
```

Parameter values might or might not be case-sensitive, it's all very individual

#### 8.3.2. Charset

HTTP uses charset names defined in RFC6365, and are case-insensitive

#### 8.3.3. Multipart Types

A sender MUST generate only CRLF to represent line breaks between body parts

### 8.4. Content-Encoding

Primarily used to indicate compression or whatever

```
Content-Encoding ::= content-coding#
```

The sender that applied the encoding(s) MUST list them in the order in which they were applied. The coding named "identity" is reserved and SHOULD NOT be included.

If the media type includes an inherent encoding, it is not restated in Content-Encoding (unless it is applied once more outside of the media type definition)

An origin server MAY respond with 415 if a representation in the request has unacceptable coding.

#### 8.4.1. Content Codings

```
content-coding ::= token
```

##### 8.4.1.1. Compress Coding

"compress" - Lempel-Ziv-Welch (LZW) coding. A recipient SHOULD consider "x-compress" to be equivalent to "compress"

##### 8.4.1.2. Deflate Coding

The "deflate" coding is a "zlib" data format, uses a combination of LZ77 and Huffman coding

##### 8.4.1.3. Gzip Coding

"gzip" - LZ77 coding with a 32-bit Cyclic Redundancy Check. A recipient SHOULD consider "x-gzip" to be equivalent to "gzip"

### 8.5. Content-Language

Content-Language header field describes the natural language(s) of the intended audience (!) for the representation

```
Content-Language ::= language-tag#
```

If not specified, the content is intended for all language audiences, for any reason

I will stress it again - Content-Language represents the target audience, not languages used in the representation.

Content-Language MAY be applied to any media type

#### 8.5.1. Language Tags

Defined in RFC5646. Only languages that humans use, strictly not programming languages.

```
language-tag ::= // Language-Tag in RFC5646
```

### 8.6. Content-Length

Content-Length header field indicates the octet length of representation's data in decimal. It can be used to delimit framing.

```
Content-Length ::= DIGIT+
```

A user agent SHOULD send Content-Length when the method defines a meaning for enclosed content and it is not sending Transfer-Encoding. For example, for a POST request a user sends Content-Length even if it is 0. A user agent SHOULD NOT send a Content-Length header if the request message isn't supposed to or doesn't have content.

A server MAY send a Content-Length header in a response to a HEAD request. A server MUST NOT do so, if it wouldn't be equal for a response to a GET request.

A server MAY send a Content-Length header field in a 304 response to a conditional GET request. It MUST NOT do so, unless this number would be equal in a 200 response

A server MUST NOT send Content-Length for 1xx and 204 responses, or 2xx responses to a CONNECT request.

Otherwise, if Transfer-Encoding is absent, an origin server SHOULD send a Content-Length header when the content size is known prior to sending.

A recipient MUST anticipate large decimal numberals and prevent parsing errors.

A sender MUST NOT forward a message if Content-Length is known/found to be incorrect.

A sender MUST NOT forward a message if Content-Length does not match the grammar. A single exception - value of Content-Length is the same decimal number repeated multiple times with commas (as a list production). In that case, sender MAY either reject the message, or fix the field value.

### 8.7. Content-Location

Content-Location references a URI to retrieve the resource corresponding to the representation in the message's content.

```
Content-Location ::= absolute-URI
                   | partial-URI // relative to the target URI
```

It is not a replacement for the target URI

If Content-Location is included in a 2xx response and it is equivalent to the target URI, the recipient MAY consider the content to be a current representation of that resource. For GET or HEAD request this is nothing new. For PUT or POST it implies that the response contains the new representation.

If Content-Location is included in a 2xx response, but it differs from target URI, then the origin server claims that the URI is an identifier to a different resource, whose representation is in the body.

- For a response to a GET or HEAD request, this means that Content-Location is a more specific identifier for the selected representation
- For a 201 response, such that Content-Location is identical to Location field value, this content is a current representation of the newly created resource
- Otherwise, it is an identifier for the same resource

A user agent that sends Content-Location in a request message is stating where it originally got that representation from.

An origin server that receives a Content-Location MUST treat the information as transitory request context, rather than as metadata. An origin server MAY use that context for request processing or whatever. An origin server MUST NOT use such information to alter the request semantics.

Example: if a client makes a PUT request, and the origin server accepts that PUT, the new state of the resource is expected to be with what was provided in the request. 

### 8.8. Validator Fields

Validator is a resource metadata that can be used within a precondition for a conditional request.

For safe requests validator fields describe the selected representation. It may refer to different representation than in response body. For a state-changing request, they describe the new representation

This spec defines modification dates and opaque entity tags

#### 8.8.1. Weak versus Strong

Validators can be weak or strong

A strong validator changes whenever a representation changes (a change which would be observable in a 200 response to GET)

It also may change when a semantically significant part of the representation metadata is changed. It should be changed only when significant, for example for cache invalidation

A strong validator is unique across all versions of all representations associated with a particular (!) resource over time.

Representation (and metadata, if significant) hashing (assuming no collisions) is a strong validator

A weak validator is the one that is not strong lol (i.e. it may be not unique, or not change every time a change occurs, etc)

An origin server SHOULD change a weak entity tag whenever it considers prior representations to be unacceptable as a substitute for the current representation (again, caching)

#### 8.8.2. Last-Modified

```
Last-Modified ::= HTTP-date
```

Self-explanatory

##### 8.8.2.1. Generation

An origin server SHOULD send Last-Modified if the date can be reasonably and consistently determined

If a resource is combined of multiple arbitrary things, the latest date of any of those things should be used

An origin server SHOULD obtain Last-Modified as close as possible to the time that it generates the Date field

An origin server with a clock MUST NOT generate Last-Modified that is later than Date, and MUST replace the Last-Modified value with Date value in such case

An origin server without a clock MUST NOT generate Last-Modified unless that date was assigned to the resource by some other system

##### 8.8.2.2. Comparison

Last-Modified in a request is assumed to be weak, unless

- The validator is being compared by an origin server to the actual current validator, and
- The origin server knows that the associated representation did not change twice during the second covered by the presented validator

or

- The validator is about to be used by a client in an If-Modified-Since, If-Unmodified-Since, If-Range header field, because the client has a cache entry, and
- The cache entry includes a Date, which is at least one second after the Last-Modified value

or

- The validator is being compared by an intermediate cache to the validator stored in its cache entry, and
- The cache entry includes a date... (copy-paste from above)

#### 8.8.3. ETag

```
ETag            ::= entity-tag
entity-tag      ::= [ weak ] opaque-tag
weak            ::= "W/"
opaque-tag      ::= '"' etagc* '"'
etagc           ::= 0x21 | 0x23-0x7E | obs-text // VCHAR - { '"' } + obs-text
```

Servers ought to avoid backslash characters in entity tags

"W/" indicates the weakness, strong by default

If a specific ETag is not generated as a strong validator, the origin server MUST mark it as weak

A sender MAY send the ETag field in a trailer section, however, sending it in the header is preferred

##### 8.8.3.1. Generation

However the hell you want, considering the weak/strong semantics

An origin server SHOULD send an ETag if it can be reasonably and consistently determined

##### 8.8.3.2. Comparison

If both are strong, a simple string equality

Otherwise, a string comparison, but excluding the "W/"

##### 8.8.3.3. Example: Entity Tags Varying on Content-Negotiated Resources

Read the original

## 9. Methods

### 9.1. Overview

```
method ::= token
```

A method indicates the purpose and client's expectations of a request

Method's semantics may be further specialized by some header fields

The method token is case-sensitive

This specification defines some of the most common methods

- GET: Get a current representation of the target resource
- HEAD: Same as GET, but do not transfer the response content
- POST: Perform resource-specific processing on the request content
- PUT: Replace all current representations of the target resource with the request content
- DELETE: Remove all current representations of the target resource
- CONNECT: Establish a tunnel to the server identified by the target resource
- OPTIONS: Describe the communication options for the target resource
- TRACE: Perform a message loop-back test along the path to the target resource

All servers MUST support GET and HEAD. Other methods are OPTIONAL

The set of supported methods for a target resource can be listed in an Allow header field

An origin server that receives an unrecognized/unimplemented method SHOULD respond with the 501. If it is recognized, but unsupported/unallowed, a server SHOULD respond with 405

### 9.2. Common Method Properties

#### 9.2.1. Safe Methods

A method is considered safe, if it is essentially read-only

A server, of course, may do unsafe and mutating behaviour, but that is strictly not client's "fault" or will. Anyways, just be reasonable

GET, HEAD, OPTIONS, TRACE methods are safe

A user agent SHOULD distinguish between safe and unsafe methods when presenting potential actions to a user

If a server supports specifying behaviour via a query or smth similar, it MUST disable or disallow such action if it is unsafe, while the method of the request is safe

#### 9.2.2. Idempotent Methods

A method is idempotent, if sending the same request multiple times is equivalent to sending it once. All safe methods, and also PUT, DELETE are idempotent

Similar nuances as with safe methods apply

A client SHOULD NOT automatically retry a request with a non-idempotent method, unless it is sure it'll be idempotent in practice, or if it can determine that the first time didn't go through

A proxy MUST NOT automatically retry non-idempotent requests. A client SHOULD NOT automatically retry a failed automatic retry

#### 9.2.3. Methods and Caching

A method needs to explicitly allow caching for caching to be employed

This specification defines caching semantics for GET, HEAD, POST.

### 9.3. Method Definitions

#### 9.3.1. GET

The GET method requests trasfer of a current selected representation for the target resource.

A client can alter the semantics of GET to be a "range request", requesting only some part(s) of the representation, by sending a Range header field

A client SHOULD NOT generate content in a GET request unless it is made directly to an origin server that has in any way indicated that such functionality is supported. An origin server SHOULD NOT rely on private agreements to receive content

The response to a GET is cacheable. A cache MAY use it to satisfy subsequent GET and HEAD requests unless otherwise indicated by the Cache-Control header

#### 9.3.2. HEAD

HEAD is identical to GET, but the server MUST NOT send content in the response. HEAD is used to only get metadata

The server SHOULD send the same headers to equivalent HEAD and GET requests. However, a server MAY omit header fields for which a value is determined only while generating the content.

A client SHOULD NOT generate content in a HEAD request unless it is made directly to an origin server that has in any way indicated that such functionality is supported. An origin server SHOULD NOT rely on private agreements to receive content

The response to a HEAD is cacheable. A cache MAY use it to satisfy subsequent GET and HEAD requests unless otherwise indicated by the Cache-Control header. A HEAD response might affect previously cached GET responses

#### 9.3.3. POST

The POST method requests that the target resource in some way processes the representation within the request.

An origin server indicates ersponse semantics by choosing an appropriate status code.

If one or more resources have been created after a POST request, the server SHOULD send a 201 response with a Location header for the primary resource created and a representation that describes the status of the request while referring to the new resource(s)

Responses to POST are only cacheable when they include explicit freshness information and a Content-Location header that has the same value as the requests target URI. A cached POST can be reused to satisfy a GET or HEAD request

If the result of processing POST would be equivalent to another resource, an origin server MAY redirect by sending 303 response

#### 9.3.4. PUT

The PUT method requests that the state of the target resource be created or replaced with the state defined by the request content. A successful PUT suggests that a subsequent GET would result in an equivalent representation in a 200 response.

If the representation has in fact been created, the origin server MUST send a 201 response. If it was modified, the origin server MUST send either a 200 or 204 response

An origin server SHOULD verify that the PUT representation is consistent with its configured constraints for the target resource. If the PUT representation is inconsistent, the server SHOULD either make it consistent, or respond with an appropriate error message (409 and 415 are suggested)

An origin server SHOULD ignore unrecognized header and trailer fields in a PUT request

An origin MUST NOT send a validator field in a successful response to PUT unless absolutely no transformation was applied to the content and the validators refer to the new representation.

PUT signifies a replacement (or creation) with the request content used as the base, while POST signifies that the resource will process that content somehow. Hence, PUT is idempotent.

A service that selects a URI on behalf of the client SHOULD be implemented using the POST method rather than PUT. If the origin server won't apply PUT to the specified resource, but wishes it was to a different one, it MUST send a 3xx response; the user agent MAY then proceed accordingly

If a PUT request is recorded in cache, responses for that target URI are to be invalidated

#### 9.3.5. DELETE

The DELETE method requests that the origin server remove the association between the target resource and its current functionality.

The representations of the target resource might or might not be actually destroyed

If a DELETE is successfully applied, the origin server SHOULD send

- a 202 if the action will likely succeed but has not yet been enacted
- a 204 if it has been enacted
- a 200 if it has been enacted and there's extra information

A client SHOULD NOT generate content in a DELETE, an origin server SHOULD NOT rely on private agreements to receive such content (same as GET and HEAD)

If a DELETE is recorded in cache, responses to that target URI are to be invalidated

#### 9.3.6. CONNECT

The CONNECT method requests that the recipient establish a tunnel to the destination origin server and, if successful, thereafter restrict its behavior to blind forwarding of data, in both directions, until the tunnel is closed

A client MUST send the port number in the request target, which consists *only* of the host and port number

A server MUST reject a CONNECT that targets an empty or invalid port number, typically with a 400

An origin server MAY accept a CONNECT request, but most do not implement it themselves (and rely on proxies)

Any 2xx response indicates that the sender (and all inbound proxies) will switch to tunnel mode immediately after the response header section. Any other response indicates that the tunnel has not yet been formed

A tunnel is closed when a tunnel intermediary detects that either side has closed its connection; the intermediary MUST attempt to send any outstanding data to the not-yet-closed side, close both connections and discard all remaining data

Proxies that support CONNECT SHOULD restrict its use to a configurable/limited set of known ports, preventing spam and stuff

A server MUST NOT send any Transfer-Encoding or Content-Length headers in a 2xx response to CONNECT. A client MUST ignore such headers

A CONNECT request message does not have content.

#### 9.3.7. OPTIONS

The OPTIONS method requests information about the communication options available for the target resource, at either the origin server or an intervening intermediary.

An OPTIONS request with an asterisk "\*" as the request target applies to the server in general rather than to a specific resource

A server generating a successful response to OPTIONS SHOULD send any header that might indicate optional features implemented by the server and applicable to the target resource, including potential extensions. The response content, if any, might include extra related stuff, not defined by this specification

A client MAY send a Max-Forwards header field in an OPTIONS to target a specific recipient in the request chain. A proxy MUST NOT generate a Max-Forwards header while forwarding, unless that request was received with Max-Forwards

A client that generates an OPTIONS request containing content MUST send a valid Content-Type describing it

#### 9.3.8. TRACE

The TRACE method requests a remote, application-level loop-back of the request message. The final recipient of the request SHOULD reflect the message received (with some exceptions) as the content of a 200 response. The "message/http" format is one way to do so. The final recepient is either the origin server, or the first server to receive Max-Forwards = 0

A client MUST NOT generate fields in a TRACE request containing sensitive data. The final recipient SHOULD exclude such fields

A client MUST NOT send content in a TRACE request

## 10. Message Context

### 10.1. Request Context Fields

The request header fields below provide additional information about the request context, including information about the user, user agent, and resource behind the request

#### 10.1.1. Expect

```
Expect      ::= expectation#
expectation ::= token [ "=" ( token | quoted-string ) parameters ]
```

The Expect header indicates a certain set of behaviours/expectations that need to be supported by the server

THe field value is case-insensitive

This specification only defines "100-continue" expectation with no defined parameters

A server that receives a non "100-continue" expectation MAY respond with a 417

A "100-continue" informs recipients that the client is about to send (presumably large) content in this request and wishes to receive a 100 interim response if the method, target URI, and header fields are not sufficient to cause an immediate success, redirect, or error response

Requirements for clients:

- A client MUST NOT generate a 100-continue expectation in a request with no content
- A client that will wait for a 100 response before sending the request content MUST send an Expect header containing a 100-continue expectation
- A client that sends a 100-continue expectation is not required to wait for any specific length of time; such a client MAY proceed to send the content anyways. A client SHOULD NOT wait for an indefinite period before sending the content
- A client that receives a 417 in response to a 100-continue SHOULD repeat that request without a 100-continue expectation

Requirements for servers:

- A server that receives a 100-continue in an HTTP/1.0 request MUST ignore that expectation
- A server MAY omit sending a 100 if it has already received some or all the content, of if there is no content
- A server that sends a 100 MUST ultimately send a final status code after processing the request content, unless the connection has been closed prematurely
- A server that responds with a final status code before reading the entire request content SHOULD indicate whether it intends to close the connection of continue reading the request content

After receiving a HTTP/1.1+ request with 100-continue and everything except the content (yet!), an origin server MUST either

- immediately respond with a final status code, or
- immediately respond with a 100 (if the final status code can't be determined just yet)

The origin server MUST NOT wait for the content before sending the 100

A proxy in such scenario MUST either

- immediately respond with a final status code (if can be determined without the content), or
- forward the request-line and header section inbounds

If proxy believes that the next inbound server only supports HTTP/1.0 it MAY immediately respond with a 100

#### 10.1.2. From

```
From    ::= mailbox
mailbox ::= // mailbox from RFC5322 Section 3.4.
```

The From header refers to the human user controlling the requesting user agent

A user agent SHOULD NOT send a From header without explicit user configuration

A robotic user agent SHOULD send a valid From header

A server SHOULD NOT use the From header for access control or authentication

#### 10.1.3. Referer

[sic]

```
Referer ::= absolute-URI | partial-URI
```

Allows to specify where the target URI was obtained, A user agent MUST NOT include the fragment and userinfo, if any, when generating Referer

If the referrer doesn't have its own URI, the user agent MUST either exclude the Referer header field or send it with a value of "about:blank"

A user agent MAY truncate parts other than the referring origin

A user agent SHOULD NOT send a Referer if the referring resource was accessed with a secure protocol and the request is not the same origin, unless that origin explicitly allows that. A user agent MUST NOT send a Referer in an unsecured HTTP request

An intermediary SHOULD NOT modify or delete the Referer when the field value shares the same scheme and host as the target URI

#### 10.1.4. TE

```
TE                  ::= t-codings#
t-codings           ::= "trailers" | ( tranfer-coding [ weight ] )
tranfer-coding      ::= token ( OWS ";" OWS transfer-parameter )*
transfer-parameter  ::= token BWS "=" BWS ( token | quoted-string )
```

TE describes capabilities of the client regarding transfer codings and trailer sections

A sender of TE MUST also send a "TE" connection option within the Connection header field

#### 10.1.5. User-Agent

```
User-Agent      ::= product ( RWS ( product | comment ) )*
product         ::= token [ "/" product-version ]
product-version ::= token
```

The User-Agent contains information about the user agent responsible for the request

A user agent SHOULD send a User-Agent unless specifically configured not to do so

"Product" refers to software responsible. Product identifiers are listed in decreasing order of their significance

A sender SHOULD limit generated product to what is necessary. A sender MUST NOT generate advertising or other redundant information there. A sender SHOULD NOT generate information in product-version not regarding the version

A user agent SHOULD NOT generate a User-Agent with too much detail and SHOULD limit the addition of subproducts by third parties.

### 10.2. Response Context Fields

Provide extra context to the response

#### 10.2.1. Allow

```
Allow ::= method#
```

Allow lists the set of methods supported by the target resource

An origin server MUST generate an Allow header in a 405 response, and MAY do so in any other response. Empty Allow is possible

A proxy MUST NOT modify the Allow header field

#### 10.2.2. Location

```
Location ::= URI-reference
```

Location may refer to a specific resource in relation to the response

If it is a relative reference, the target URI is used as the base point

For 201 response Location refers to the primary resource created by the request. For 3xx, the Location refers to the preferred target resource

If the Location in a 3xx response does not have a fragment component, a user agent MUST attach the original target URI fragment (if there was one)

#### 10.2.3. Retry-After

```
Retry-After     ::= HTTP-date | delay-seconds
delay-seconds   ::= DIGIT+
```

"Retry-After" indicates how long the user ought to wait before making a follow-up request. When sent in a 503 response, Retry-After indicates how long the service is expected to be unavailable to the client. When sent with any 3xx response - the minimum time to wait before issuing a redirect request

#### 10.2.4. Server

```
Server ::= product ( RWS ( product | comment ) )*
```

Indicates software used by the server to handle the request. An origin server MAY generate a Server header in its responses

Similar points as with User-Agent apply (SHOULD NOT too long, SHOULD limit third party additions)

## 11. HTTP Authentication

### 11.1. Authentication Scheme

```
auth-scheme ::= token -- case insensitive
```

This specification defines the scheme framework, not any schemes themselves

A server can challenge a client, and client can provide authentication information

### 11.2. Authentication Paramaters

```
token68     ::= (ALPHA | DIGIT | "-" | "." | "_" | "~" | "+" | "/")+ "="*
auth-param  ::= token BWS "=" BWS ( token | quoted-string )
```

Each auth-param MUST only occur once per challenge

### 11.3. Challenge and Response

```
challenge   ::= auth-scheme [ SP+ ( token68 | auth-param# ) ]
```

A 401 response is used to challenge the autorization of a user agent, including a WWW-Authenticate field with at least one challenge applicable to the requested resource

A 407 response is used for the same thing, but by a proxy, with a Proxy-Authenticate field

Listing commonly supported schemes such as "basic" first is advised

To address a 401, or pre-emtively, user agent can include an Authorization header with the request

To address a 407 - Proxy-Authorization header

### 11.4. Credentials

```
credentials ::= auth-scheme [ SP+ ( token68 | auth-param# ) ]
```

The user agent selects the most secure (as it sees) auth-scheme that it understands from the challenge, and provides user credentials. Transmission of credentials is a significant security consideration

A server SHOULD respond with a 401 with WWW-Authenticate header (with at least one challenge, possibly new ones) if it receives partial/invalid/none credentials to a protected resource

A proxy SHOULD do the same - respond with 407 and Proxy-Authenticate

A server that receives valid, but not fitting credentials, ought to respond with a 403

### 11.5. Establishing a Protection Space (Realm)

The "realm" authentication parameter is reserved for use by authentication schemes that wish to indicate a scope of protection

A realm is pretty much a semantic partitioning of resources with regards to authentication and security and stuff. The realm value is a string

If a prior request has been authorized, the user agent MAY reuse the same credentials for all other requests within that protection space for a period of time determined by the authentication scheme, parameters, and/or user preferences

A sender MUST only generate the quoted-string syntax

### 11.6. Authenticating Users to Origin Servers

#### 11.6.1. WWW-Authenticate

```
WWW-Authenticate ::= challenge#
```

Indicates auth scheme(s) and parameters applicable to the target resource

A server generating a 401 MUST send a WWW-Authenticate with at least one challenge. A server MAY generate a WWW-Authenticate in other responses to indicate that supplying credentials might affect the response

A proxy forwarding a response MUST NOT modify any WWW-Authenticate

Only one challenge is advised to be sent at a time. User agents must take special care when parsing WWW-Authenticate for more than one challenge.

Spec notes that there might be an empty entry to the list of challenges, but I don't see where the grammar allows this???

#### 11.6.2. Authorization

```
Authorization ::= credentials
```

Allows user agent to authenticate itself (possibly after receiving a 401).

Valid credentials for a specific realm are presumed to work for all requests within that realm

A proxy MUST NOT modify any Authorization header fields

RFC9111 Section 3.5.

#### 11.6.3. Authentication-Info

```
Authentication-Info ::= auth-param#
```

Server can communicate information after client's authentication credentials have been accepted

Can be used in any HTTP response. Semantics are defined by the authentication scheme indicated by the Authorization header of the corresponding request

A proxy must not (MUST NOT? not specified) modify

Can be sent as a trailer field if the authentication scheme explicitly allows

### 11.7. Authenticating Clients to Proxies

#### 11.7.1. Proxy-Authenticate

```
Proxy-Authenticate ::= challenge#
```

A proxy MUST send at least one Proxy-Authenticate with at least one challenge in each 407 response

Applies only to the next outbound client on the response chain, but it is common to use if multiple chained proxies are used. In such configuration, Proxy-Authenticate will be forwarded until consumed

Stuff similar to WWW-Authenticate applies about parsing

#### 11.7.2. Proxy-Authorization

```
Proxy-Authorization ::= credentials
```

Applies only to the next inbound proxy that demanded with a Proxy-Authenticate. A proxy MAY relay the credentials to the next proxy, if configured

#### 11.7.3. Proxy-Authentication-Info

```
Proxy-Authentication-Info ::= auth-param#
```

Same as Authentication-Info, but for proxy authentication

Applies only to the next outbound client, read Proxy-Authenticate

## 12. Content Negotiation

Ways for a client and a server to negotiate the content lol

Three patterns of content negotiation:

- Proactive - server selects a representation based on client's stated wishes
- Reactive - client requests a list of available options from the server
- Request content - a request with an option from the list received from reactive

There is also "conditional content" - selectively rendered based on user parameters, "active content" - contains a script that makes additional (more specific) requests based on the user agent characteristics, and "Transparent Content Negotiation" [RFC2295], where selection is performed by an intermediary. These are not mutually exclusive

### 12.1. Proactive Negotiation

Preferences are sent by the user agent in a request (also known as "server-driven negotiation").

A user agent MAY send request header fields that describe its preferences

Disadvantages

- Server can't decide for sure what's the best for the client
- Having the user describe its capabilities in every request can be inefficient and a privacy risk
- Complicates the origin server implementation
- Limits caching responses

A Vary header field is often send in a response to proactive negotiation to indicate what parts of the request were used

Header fields Accept, Accept-Charset, Accept-Encoding, Accept-Language are used by user to proactively negotiate. These apply to any content in the response, including target resource, errors, etc

### 12.2. Reactive Negotiation

User selects content after receiving an initial response

If the user is not satisfied by the initial response, it can perform a GET on one or more of the alternative resources

A server might choose not to send an initial representation, only the list. 300 and 406 are often used for this

Advantageous when the server can't easily decide, the user knows better

Disadvantageous cuz more latency (at least 2 requests)

### 12.3. Request Content Negotiation

Preferences listed by a user in request, as a response to reactive negotiation, are called "request content negotiation". Accept and Accept-Encoding can be sent.

### 12.4. Content Negotiation Field Features

#### 12.4.1. Absence

Absence of a field implies lack of sender's preference in that aspect

If present, but none representations can be considered acceptable, server can either respond 406, or disregard the header field

#### 12.4.2. Quality Values

```
weight  ::= OWS ";" OWS "q=" qvalue
qvalue  ::= ( "0" [ "." [ DIGIT [ DIGIT [ DIGIT ] ] ] ] )
          | ( "1" [ "." [ "0" [ "0" [ "0" ] ] ] ] )
```

"q" - case insensitive, used to specify relative preference

Weight is between 0 and 1, where 0.001 is the least preferred, 1 is the most preferred, 0 is "not acceptable". The default weight is 1

A sender of a qvalue MUST NOT generate more than 3 digits after the "."

#### 12.4.3. Wildcard Values

Most of these headers define a wildcard "\*" to select unspecified values If no wildcard is present, not mentioned values are considered unacceptable. Within Vary, "\*" means that the variance is unlimited

### 12.5. Content Negotiation Fields

#### 12.5.1. Accept

```
Accept      ::= ( media-range [ weight ] )#
media-range ::= ( (  "*" "/" "*" )
                | ( type "/" "*" )
                | ( type "/" subtype ) )
                parameters
```

"\*" is used to group media types into ranges. "\*/\*" - all, "type/\*" - all subtypes of type

Each media-range may be followed by parameters, with optional "q" as the last one indicating the preference. Senders SHOULD send "q" last (if at all). Recipients SHOULD process any parameter named "q" as weight, regardless of order

If more than one range applies to a given type, the most specific takes precedence

#### 12.5.2. Accept-Charset [DEPRECATED]

```
Accept-Charset  ::= ( ( token | "*" ) [ weight ] )#
```

Indicates user agent's preference for charsets in textual response content

A user agent MAY associate a quality value

"\*" is a special value, matching every charset not mentioned elsewhere

Deprecated, because everyone reasonable should be using UTF-8

#### 12.5.3. Accept-Encoding

```
Accept-Encoding ::= ( codings [ weight ] )#
codings         ::= content-coding | "identity" | "*"
```

Indicates preferences regarding the use of content codings

When sent in a request, indicates codings acceptable in response

When send in a response, provides information which codings are preferred in a subsequent request to the same resource

Each value MAY have quality. "\*" matches every not mentioned elsewhere

A server applies the following logic

1. If no Accept-Encoding is in the request, any is considered acceptable
2. If the representation has no content coding, then it is acceptable, unless stuff like "identity;q=0" or "\*;q=0", provided no more specific entries
3. If is listed, then acceptable, unlless q=0

When selecting between multiple content codings that have the same purpose, the highest non-zero qvalue is preferred

Empty Accept-Encoding implies that user does not want any coding in the response. If no coding is acceptable, the origin server SHOULD send a response without any content coding, unless "identity" is indicated as unacceptable

Accept-Encoding in a response indicates what codings the specific resource was willing to accept in the associated request (at that time)

If server fails due to unsupported coding it ought to respond with a 415 and include an Accept-Encoding. Servers that fail with a 415 for reasons unrelated to content codings MUST NOT include a Accept-Encoding

Most commonly used in 415 in response to optimistic use of a content coding by clients. Can also indicate that content codings are supported to optimize future interactions, i.e. might include in a 2xx when the request content was big enough to justify use of compression coding

#### 12.5.4. Accept-Language

```
Accept-Language ::= ( language-range [ weight ] )
language-range  ::= // language-range, RFC4647 Section 2.1.
```

Indicates the set of natural languages that are preferred in the response (view Section 8.5.1.)

Each can have quality value

Users should, along with assigning decreasing qvalues, list items in decreasing order, due to some erroneous implementations (also view RFC4647 Section 2.3.)

Language matching - RFC4647 Section 3

User agent's language selection should be configurable. A user agent now allowing configuration MUST NOT send an Accept-Language

#### 12.5.5. Vary

```
Vary ::= ( "*" | field-name )#
```

In a response describes what part (or lack of it) of a request might have influenced the server's content selection process

"\*" in the list indicates there might be more things that have influenced that. A proxy MUST NOT generate a "\*" in a Vary

A Vary field has two purposes

1. Informs cache that they MUST NOT use this response unless specified values are equivalent
2. Informs user agent that content negotiation is going on and what might affect it

An origin server SHOULD generate a Vary on a cacheable response when it wants to

Vary might be elided when considered insignificant by the server

Authorization should not be sent in Vary
Apparently in Section 11.6.2 it is stated that "reuse of that response is prohibited by the field definition", and I don't see how that's the case

## 13. Conditional Requests

A conditional HTTP request indicates a precondition to be tested before touching the target resource

Conditional GET requests are frequently used for cache updates

### 13.1. Preconditions

Precondition compares a set of previously received validators to their current state for the selected representation.

#### 13.1.1. If-Match

```
If-Match ::= "*" | entity-tag#
```

Checks if there is at least one representation if "\*", or current representation's entity tag matches at least one in the list

An origin server MUST use the strong comparison function for If-Match

Frequently used to check, if the current representation is still what the client thinks it is, and hasn't been updated recently

If receive If-Match the server MUST do

1. If value is "\*", the condition is true if current representation exists
2. If is a list, if any tag matches the current, true
3. Otherwise false

If If-Match evaluated false the server MUST NOT perform the requested method. It MAY indicate failure by responding 412. It the requests seems to apply the same change that has already been done, the server MAY respond with a 2xx

A client MAY send an If-Match if it wants its main functionality (???)

A cache or intermediary MAY ignore If-Match

#### 13.1.2. If-None-Match

```
If-None-Match ::= "*" | entity-tag#
```

No current representation if "\*", or current representation doens't match any listed

A recipient MUST use the weak comparison

When updating cached responses a client SHOULD generate an If-None-Match. Server can respond with 304

"\*" prevents modification if doesn't exist

Evaluation: 

1. If "\*", false if there is current representation
2. If list, false if one of the listed matches the current
3. Otherwise true

If evaluates to false a server MUST NOT perform the requested method. It MUST respond with either a 304 to GET, HEAD, or 412 to all other requests

#### 13.1.3. If-Modified-Since

```
If-Modified-Since ::= HTTP-date
```

Checks if selected representation's modification date is later than provided

A recipient MUST ignore If-Modified-Since if there is also If-None-Match

A recipient MUST ignore If-Modified-Since if the HTTP-date is invalid, if there are multiple values, or if the method not GET or HEAD

A recipient MUST ignore if resource does not have a modification date

A recipient MUST interpret an If-Modified-Since value in terms of the server's clock

Used for

- cache, if ETag is not available
- looking for recent changes (duh)

If includes a If-Modified-Since without If-None-Match, the server SHOULD evaluate the If-Modified-Since condition per Section 13.2.

Evaluation:

1. If selected representation's date is earlier or equal to provided, false
2. Otherwise, true

An origin server SHOULD NOT perform the requested method if the conditioon evaluates to false. It SHOULD generate a 304 response, including only those metadata that are useful for identifying or updating a previously cached response

#### 13.1.4. If-Unmodified-Since

```
If-Unmodified-Since ::= HTTP-date
```

Can you guess what this means

A recipient MUST ignore If-Unmodified-Since if If-Match is present.

A recipient MUST ignore if not valid HTTP-date, or list of dates

A recipient MUST ignore if resource doesn't have modification date

A recipient MUST interpret timestamp in terms of server's clock

All of these are using for fucking cache and "lost update" problem, we get it

If If-Unmodified-Since and no If-Match, the server MUST evaluate If-Unmodified-Since as per Section 13.2.

Evaluate

1. If selected representation's date is earlier or equal to provided, true
2. Otherwise, false

If evaluates to false the server MUST NOT perform the requested method. It MAY respond with 412. If is a state-changing, and the same change has already been applied, it MAY respond with 2xx

A client MAY send If-Unmodified-Since if it wants to

A cache or intermediary MAY ignore If-Unmodified-Since

#### 13.1.5. If-Range

```
If-Range ::= entity-tag | HTTP-date
```

Instructs to ignore the Range header if the validator doesn't match

A client MUST NOT generate If-Range if there's no Range. A server MUST ignore an If-Range if there's no Range, or if target support doesn't support Range

A client MUST NOT generate an If-Range with a weak entity tag. A client MUST NOT generate it with HTTP-date unless it has no entity tag and the date is a strong validator

A server MUST evaluate as per Section 13.2.

Evaluating (with HTTP-date)

1. If validator is not strong (Section 8.8.2.2.), the condition is false
2. If matches exactly the Last-Modified for the selected representation, true
3. Otherwise, false

Evaluating (with entity-tag)

1. If exactly matches the ETag using only strong comparison, true
2. Otherwise, false

A recipient of If-Range MUST ignore the Range if If-Range evaluates to false. Otherwise, it SHOULD process the Range as requested

### 13.2. Evaluation of Preconditions

#### 13.2.1. When to Evaluate

Except when excluded, a recipient cache or origin server MUST evaluate received request preconditions after successfully performing its normal request checks and just before it would process the request content (if any), or perform the action associated with the request method. A server MUST ignore all received preconditions if its response to the same request without those conditions, prior to processing the request content, would not have been a 2xx or 412.

A server that is neither the origin server of the resource nor a cache MUST NOT evaluate the conditional request defined by this spec, and it MUST forward them if the request is forwarded. A server MUST ignore if method does not involve the selection or modification of a selected representation (CONNECT, OPTIONS, TRACE...)

#### 13.2.2. Precedence of Preconditions

The order matters

A recipient cache or origin server MUST evaluate preconditions defined here in the following order

1. When If-Match present, evaluate
    true -> step 3
    false -> respond as per Section 13.1.1.
2. When If-Match not present, and If-Unmodified-Since present, evaluate
    true -> step 3
    false -> respond as per Section 13.1.4.
3. When If-None-Match present, evaluate
    true -> step 5
    false -> respond as per Section 13.1.2.
4. When method is GET/HEAD, If-None-Match not present, If-Modified-Since present, evaluate
    true -> step 5
    false -> respond 304 (as per Section 13.1.3. ?)
5. When method is GET and Range and If-Range present, evaluate If-Range
    proceed as per Section 13.1.5.
6. Otherwise
    perform the requested method as usual

Any extensions ought to define order for evaluating their fields that are related to these

## 14. Range Requests

Used to retrieve only a part(s) of representation, for example, if you already have other parts

Range requests are an OPTIONAL feature

### 14.1. Range Units

```
range-unit ::= token
```

I guess it's units in which range is measured in? It's not explained

But at least it's case-insensitive

#### 14.1.1. Range Specifiers

```
ranges-specifier        ::= range-unit "=" range-set
range-set               ::= range-spec#+
range-spec              ::= int-range | suffix-range | other-range

int-range               ::= first-pos "-" [ last-pos ]
first-pos               ::= DIGIT+
last-pos                ::= DIGIT+

suffix-range            ::= "-" suffix-length
suffix-length           ::= DIGIT+

other-range             ::= ( 0x21-0x2B | 0x2D-0x7E )+ // VCHAR without ','
```

Range is expressed by a unit and a specified

int-range - range measured in units

int-range is invalid if the last-pos is present and less than the first-pos

suffix-range - last N units

other-range - whatever

A ranges-specifier is invalid if it contains any invalid/undefined range-spec

A valid ranges-specified is "satisfiable" if it contains at least one satisfiable range-spec (as defined per range-unit)

#### 14.1.2. Byte Ranges

The "bytes" range unit measures bytes (octets). Byte ranges do not use the other-range specified

int-range is inclusive, byte offsets start at zero

If coding is applied, the coded bytes are counted, not the original undecoded

If last-pos is absent or greater than the actual name, the server fixes it to (length - 1)

suffix-range - last N bytes (N > 0), or the whole representation if N >= length

For a GET request, a valid bytes is satisfiable if it's either

- an int-range with a first-pos < length
- a fuxxix-range with a non-zero suffix-length

Recipients MUST ancitipate potentially large decimal numerals

### 14.2. Range

```
Range ::= ranges-specifier
```

Modifies GET request to request only the specified subranges

A server MAY ignore Range

A server MUST ignore a Range if it's undefined for the specified method (or if it is unknown). This spec only defines behavior for GET

A server MUST ignore a Range if range-unit is unknown. A proxy MAY discard it if same

A server MAY ignore if invalid ranges-specifier, more than two overlapping ranges, many small ranges not in ascending order. A client SHOULD NOT be inefficient with multiple ranges

A server MAY ignore a Range when the selected representation has no content

A client SHOULD list ranges in ascending order, unless there's a specific reason.

Range is evaluated after precondition headers (Section 13.1.), and only if the response would be 200 without it.

If all is fine, the server SHOULD send a 206 response with content containing one or more partial representations that correspond to the satisfiable range-spec(s) requested

If everything is fine, but range-unit is not supported, or ranges-specified is unsatisfiable, the server SHOULD respond with 416

### 14.3. Accept-Ranges

```
Accept-Ranges       ::= acceptable-ranges
acceptable-ranges   ::= range-unit#+
```

In a response indicates whether the ranges is supported for the target resource

A client MAY generate range requests without receiving Accept-Ranges prior

A client MUST NOT assume that Accept-Ranges is persistent

A server not supporting ranges MAY send

```
Accept-Ranges: none
```

The Accept-Ranges MAY be sent in a trailer section

### 14.4. Content-Range

```
Content-Range       ::= range-unit SP ( range-resp | unsatisfied-range )
range-resp          ::= incl-range "/" ( complete-length | "*" )
incl-range          ::= first-pos "-" last-pos
unsatisfied-range   ::= "*/" complete-length
complete-length     ::= DIGIT+
```

Sent in a single part 206 response to indicate the range it has in content, sent in 416 for general information

If a 206 response with Content-Range has unit that recipient does not understand it MUST NOT attempt to recombine it with a stored representation. A proxy SHOULD forward such message downstream

Content-Range might be used for a partial PUT (Section 14.5.). A server MUST ignore a Content-Range if it's not defined for the method

For bytes, a sender SHOULD indicate the complete length, unless it is unknown or difficult to determine, indicated by "\*"

Invalid if last-pos less than first-pos, or complete-length is less or equal to last-pos. Recipient of invalid Content-Range MUST NOT attempt to recombine

A server generating 416 to bytes SHOULD send Content-Range with unsatisfied-range

Content-Range has no meaning for status codes unless explicitly specified (only 206 and 416 in this specification)

### 14.5. Partial PUT

PUT request with a Content-Range may indicate replacing within range, but is not widely supported, usually depends on private agreements with user agents

An origin server SHOULD respond with a 400 if PUT with Content-Range, unless it supports it

Not backwards compatible

### 14.6. Media Type multipart/byteranges

206 response parts are transmitted as body parts in a multipart message body (RFC2046 Section 5.1.) with the media type "multipart/byteranges"

"multipart/byteranges" includes one or more body parts, each with Content-Type and Content-Range

Implementation Notes:

- Additional CRLFs might precede the first boundary string in the body
- Occasionally quoted boundary strings are handled incorrectly
- Some implementations use "multipart/x-byteranges", not entirely compatible

Despite the name, not limited to byte ranges

Read the spec for additional details

## 15. Status Codes

Three-digit code describing the result of the request and semantics of the response. All valid codes are in [100..599] range, inclusive

- 1xx Informational: The request was received, continuiing process
- 2xx Successful: The request was successfully received, understood, and accepted
- 3xx Redirection: Further action needs to be taken in order to complete the request
- 4xx Client Error: The request contains bad syntax or cannot be fulfilled
- 5xx Server Error: The server failed to fulfill an apparently valid request

A client MUST understand the class of any status code. If status code is unknown, treate last two digits as 0 (i.e. unknown 471 -> known 400)

A client that receives an invalid (outside of range) status code SHOULD treat it as a 5xx status code

A single request may have multiple corresponding responses - many or none 1xx, followed by exactly one non-1xx.

### 15.1. Overview of Status Codes

Phrases are recommendations

Cacheable responses (200, 203, 204, 206, 300, 301, 308, 404, 405, 410, 414 an 501 per this spec, but extendable) can be reused by a cache unless otherwise indicated by the method or cache controls

### 15.2. Informational 1xx

An interim response for communicating connection status or request progress. A sender MUST NOT send a 1xx to an HTTP/1.0 client

A 1xx response is terminated by the end of the header section

A client MUST be able to parse one or more 1xx responses received prior to a final response, even if the client does not expect one. A user agent MAY ignore unexpected 1xx responses

A proxy MUST forward 1xx responses, unless proxy itself requested its generation

#### 15.2.1. 100 Continue

The initial part of a request has been received and has not yet been rejected. The server intends to send a final response after the request has been fully received and acted upon

If used with Expect, the client ought to continue sending the request, discard the 100

If no Expect, client can discard

#### 15.2.2. 101 Switching Protocols

Willing to comply with client's request (via Upgrade) to change protocol. THe server MUST generate an Upgrade header in the response that indicates which protocol(s) will be used after this response

It is assumed that server only agrees to change protocols when it is advantageous

### 15.3. Successfull 2xx

Request was successfully received, understood, and accepted

#### 15.3.1. 200 OK

Success

Meaning of content:

- GET: the target resource
- HEAD: the target resource, without representation data
- POST: the status of, or results obtained from the action
- PUT, DELETE: the status of the action
- OPTIONS: communication options for the target resource
- TRACE: the request message as received by the server returning the trace

Aside from CONNECT, expected to contain content unless framing indicates lack of it. If request somehow wishes for no content, 204 is ought to be sent

Cacheable, unless otherwise indicated

In 200 to GET or HEAD, an origin server SHOULD send any available validator fields for the selected representation, with both a strong entity tag and a Last-Modified date being preferred

In 200 to state-changing methods, any validator fields refer to the new representation

#### 15.3.2. 201 Created

Success, one or more new resources have been created. The primary is identified by either a Location, or if there is none, by the target URI

Any validator fields refer to a new representation

#### 15.3.3. 202 Accepted

Request accepted for processing, but it has not been completed

#### 15.3.4. 203 Non-Authoritative Information

Success, but the enclosed content has been modified by a transforming proxy

Cacheable, unless otherwise indicated

#### 15.3.5. 204 No Content

Success, no additional content

Implies that the user agent does not need to change its "document view", if any (e.g. "save" button)

Terminated by the end of the header section

Cacheable, unless otherwise indicated

#### 15.3.6. 205 Reset Content

Success, but desires that the "document view" is reset as it was before the request (e.g. clear out a form)

A server MUST NOT generate content in a 205 response

#### 15.3.7. 206 Partial Content

Successfully fulfill a range request by transferring one or more parts of the selected representation

A client MUST inspect a 206 response's Content-Type and Content-Range to determine which ranges have been satisfied

A server generating a 206 MUST generate the following header fields, if they would've appeared in a 200 response - Date, Cache-Control, ETag, Expires, Content-Location, Vary, and those required in the subsections below

Content-Length indicates length of a part

A server responding 206 to request with If-Range SHOULD NOT generate other representation headers beyond those required. Otherwise a sender MUST generate all headers that would have been sent in a 200

Cacheable, unless otherwise indicared

##### 15.3.7.1. Single Part

If single part, the server generating 206 MUST generate a Content-Range

##### 15.3.7.2. Multiple Parts

If multiple parts, the server generating 206 MUST generate "multipart/byteranges" content, as per Section 14.6., and a Content-Type header containing "multipart/byteranges". A server MUST NOT generate Content-Range if multiple parts (will be sent in each part instead)

Within the header of each body part the server MUST generate a Content-Range. If there would be a Content-Type in a 200, the server SHOULD generate Content-Type in each body part

When multiple ranges, a server MAY coalesce those that overlap, or if sending a bigger range would be more efficient

A server MUST NOT generate a multipart response to a request for a single range. It MAY generate a "multipart/byteranges" with only one part if multiple ranges were requested. A client that cannot process "multipart/byteranges" MUST NOT request for multiple ranges

A server generating a multipart response SHOULD send the parts in the same order as requested, excluding unsatisfieble or coalesced ones. A client receiving a multipart response MUST inspect Content-Range in each body part to determine which one it is

##### 15.3.7.3. Combining Parts

Client MAY combine ranges received from GET iff the strong validator matches

The most recent 200 response's headers are used for the final recombination. If no 200 were received (only 206), the most recent out of them s used. The client MUST use other header fields in the new response, aside from Content-Range, to replace all instances of the corresponding header fields in the stored response

If the whole representation has been recombined, the client MUST treat it as it has received a 200, including a Content-Length. Otherwise, it  MUST process either as an incomplete 200 (if prefix), a single 206 with "multipart/byteranges" content, or multiple 206 each with single range

### 15.4. Redirection 3xx
