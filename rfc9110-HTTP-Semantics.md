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

For safe requests validator fields describe the selected representation. It may refer to different representation than in response body


