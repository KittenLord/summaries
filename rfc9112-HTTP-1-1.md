Source: https://www.rfc-editor.org/rfc/rfc9112.pdf

# RFC 9112 - HTTP/1.1

## Abstract

HTTP is HTTP

## 1. Introduction

HTTP/1.1 is defined by this document, RFC9110 and RFC9111

### 1.1. Requirements Notation

RFC 2119 RFC8174

### 1.2. Syntax Notation

Originally ABNF, I'll use whatever I come up with

'#' denotes a comma-separated list, analogous to '*'

## 2. Message

RFC9110 Section 3

### 2.1. Message Format

```
HTTP-message    ::= start-line CRLF
                    (field-line CRLF)*
                    CRLF
                    [message-body]

start-line      ::= request-line
                  | status-line
```

A message is either a request from client or a response from server. They only differ in start-line (status-line for the response).

### 2.2. Message Parsing

1. Read the start-line
2. Read each header field line until the empty CRLF line
3. Determine if there is a body, or what length it is
4. Read body, until enough bytes are read, or connection is closed

A recipient MUST parse an HTTP message as if it were using an encoding that is a superset of US-ASCII, not Unicode or whatever.

A recipient MAY recognize a single LF as a line terminator and ignore any preceding CR

A sender MUST NOT generate a CR without a following LF anywhere outside the content. A recipient of such atrocity MUST either completely invalidate, or fix the message by replacing each bare CR with SP

An HTTP/1.1 user agent MUST NOT preface or follow a request with an extra CRLF. A trailing CRLF MUST be accounted for in message body length

A server SHOULD ignore at least one empty CRLF prior to the request-line

A sender MUST NOT send whitespace between start-line and the first header field. A recipient of such an invalid message MUST either completely invalidate the message, or ignore each whitespace-preceded line without any processing of it until encountering a first valid header field.

If a server accepts something that doesn't look like HTTP, it SHOULD    respond with 400 response and close the connection.

### 2.3. HTTP Version

```
// case-sensitive
HTTP-version    ::= "HTTP" "/" DIGIT "." DIGIT
```

If the recipient uses HTTP/1.0 or unknown version, HTTP/1.1 sender sends a message that can be interpreted as a valid HTTP/1.0 message, provided that all newer features are ignored.

Intermediaries MUST send their own HTTP-version in forwarded messages, unless it is purposefully downgraded.

A server MAY send an HTTP/1.0 response to an HTTP/1.1 request if the client is suspected to be incorrectly implemented. Such downgrades SHOULD NOT be performed unless there's a solid reason to

## 3. Request Line

```
request-line    ::= method SP request-target SP HTTP-version
```

Recipients may MAY parse SP as if it were RWS (except for CRLF), where RWS contains SP, HTAB, VT, FF, bare CR

A server that receives a method longer than any that it implements SHOULD respond with 501. Similar with request-line, but MUST respond with 414.

It is recommended to support request-line at least 8000 octets long.

### 3.1. Method

```
method ::= token
```

RFC9110 Section 9

### 3.2. Request Target

```
request-target  ::= origin-form
                  | absolute-form
                  | authority-form
                  | asterisk-form

origin-form     ::= absolute-path [ "?" query ]

absolute-form   ::= absolute-URI

authority-form  ::= uri-host ":" port

asterisk-form   ::= "*"
```

The request-target identifies the target resource.

No whitespace is allowed in the request-target.

Recipients of an invalid request-line SHOULD respond with 400, or 301 redirect to a properly encoded request-line, and any autocorrect SHOULD NOT occur otherwise.

A client MUST send a Host header field in all request messages. If target URI includes an authority component, then a client MUST send an identical value in Host, excluding [ userinfo "@" ]. If authority is missing or is undefined, Host MUST be empty.

A server MUST respond with a 400 to any request that lacks a Host header, has more than one, or has invalid value.

#### 3.2.1. origin-form

When making a request directly to an origin server, excluding CONNECT and OPTIONS requests, a client MUST send only the absolute path and query components of the target URI as request-target. Empty path MUST be sent as "/". A Host header is also sent (RFC9110 Section 7.2.)

#### 3.2.2. absolute-form

When making a request to a proxy, excluding CONNECT and OPTIONS requests, a client MUST send absolute-form as the request-target

RFC9110 Section 7.6.

A client MUST send a Host header field even with absolute-form request-target.

When receiving absolute-form, a proxy MUST ignore Host header field (if any), and (especially if forwarding) MUST replace it with that's in request-target.

When an origin server receives an absolute-form request, it MUST ignore received Host header and use request-target info instead.

A server MUST accept absolute-form

// NOTE: apparently proxy doesn't send absolute-form requests, but I'm not sure where it is specified

#### 3.2.3. authority-form

It is used for CONNECT requests, MUST be used when establishing a tunnel. Default scheme port is used if not specified otherwise

#### 3.2.4. asterisk-form

MUST be used for OPTIONS request

If path is empty and doesn't have query, the last proxy MUST replace request-target with "\*" for forwarded requests

### 3.3. Reconstructing the Target URI

If request-target is absolute-form, that is the target URI.

Otherwise

- If a scheme if provided by a configuration or by a trusted outgoing gateway, that scheme is used for the target URI. Otherwise, if secured connection, "https". Otherwise, "http"
- If request-target is authority-form, get authority from there. Otherwise, try getting Host header. If there is none, or it is empty/invalid, the target URI's authority is empty
- If request-target is authority-form or asterisk-form, path and query are empty. Otherwise, request-target is used.
- These components are then used to reconstruct an absolute-URI

If authority is empty, but scheme requires it to be non-empty, the server can reject the request, or determine if it can use the default, and replace it if it can before processing further.

This may, however, be unsafe, if authority that user intended doesn't match with the one determined.

RFC9110 Section 7.4.

## 4. Status Line

```
status-line     ::= HTTP-version SP status-code SP [ reason-phrase ]
status-code     ::= DIGIT DIGIT DIGIT
reason-phrase   ::= (HTAB | SP | VCHAR | obs-text)+
```

Recipients MAY parse status-line as if it was delimited by whitespace, rather than single SP (aside from CRLF). Such whitespace is SP, HTAB, VT, FF, bare CR.

Depending on status-code recipient parses the rest of the message as defined by that status code, or, if it is not recognized, by the class of the status code

A client SHOULD ignore the reason-phrase content. A server MUST send the space after status-code

## 5. Field Syntax

```
field-line      ::= field-name ":" OWS field-value OWS
```

### 5.1. Field Line Parsing

The contents of a field line value are parsed at the later stages of message interpretation

A server MUST reject with a 400 response any message which has whitespace before the ":". A proxy MUST remove such whitespace before forwarding the message.

Including a single SP after the ":" is advised.

### 5.2. Obsolete Line Folding

```
obs-fold    ::= OWS CRLF RWS
```

It was formerly used to extend a field value to the next line. Now this is forbidden except in "message/http" media type

A sender MUST NOT generate a message with obs-fold (except "message/http")

A server MUST reject with 400 (preferrably explaining the reason in the representation) or replace each obs-fold with SP+ before using the message (except "message/http")

A proxy MUST either discard the message with 502 (again preferrably with explanation) or replace ech obs-fold with SP+ (except "message/http")

A user MUST replace each obs-fold with SP+ (except "message/http")

## 6. Message Body

```
message-body        ::= OCTET*
```

The presence of a message body in a request is signaled by the Content-Length or Transfer-Encoding header field

The presence of a message body in a response depends on the request method and response status code

### 6.1. Transfer-Encoding

```
Transfer-Encoding   ::= transfer-coding# // RFC9110 Section 10.1.4.
```

A recipient MUST be able to parse the chunked transfer coding. A sender MUST NOT apply the chunked transfer coding more than once to a message body. If any other coding other than chunked is applied to a request's content, the sender MUST apply chunked as the final transfer coding. If any transfer coding other than chunked is applied to a response's content, the sender MUST either apply chunked as the final transfer coding or terminate the message by closing the connection.

Transfer-Encoding is a property of the message, not of the representation. Any recipient in the chain MAY decode the received transfer codings, or apply additional ones to the message body, assuming Transfer-Encoding gets updated. 

Transfer-Encoding MAY be sent in a response to HEAD or in 304 response to GET to indicate which transfer codings the server *would have* applied in the successful outcome.

A server MUST NOT send a Transfer-Encoding in 1xx, 204 responses; in 2xx response to a CONNECT request.

A server that receives a request message with a transfer coding it does not understand SHOULD respond with 501

It is to be assumed that implementations advertising only HTTP/1.0 do not support Transfer-Encoding, even if one forwards the header field

A client MUST NOT send Transfer-Encoding unless it knows the server will handle at least HTTP/1.1 requests. A server MUST NOT do the same with respect to the client

Transfer-Encoding overrides Content-Length if both are used.

A server MAY reject the request if both Transfer-Encoding and Content-Length are used, or consider only Transfer-Encoding. It MUST close the connection after responding to such a request.

A receiver of an HTTP/1.0 message with a Transfer-Encoding header MUST treat that message as if the framing is faulty, even if a Content-Length is present, and close the connection after processing the message.

### 6.2. Content-Length

Content-Length can provide the anticipated size of potential content, in decimal amount of octets. For messages that don't have content, Content-Length indicates the size of the selected representation

A sender MUST NOT send a Content-Length if Transfer-Encoding is present

### 6.3. Message Body Length

The length of a message body is determined by:

1. Any response to a HEAD request, or with a 1xx, 204, 304 status code is always terminated after the first empty line

2. Any 2xx response to a CONNECT request implies the tunnel immediately after the first empty line. The client MUST ignore Content-Length and Transfer-Encoding in such message

3. If both Transfer-Encoding and Content-Length are present, former is used, though such a message is ought to be handled as an error. An intermediary that still chooses to forward the message MUST remove Content-Length and process Transfer-Encoding (below)

4. If Transfer-Encoding is present and chunked transfer encoding is the final encoding, the message body length is determined by reading and decoding the chunked data until the transfer coding indicates the data is complete

If, in a response, chunked is not the final encoding, length is determined by reading the connection until it's closed by the server

If, in a request, same thing, the server MUST respond with 400 and close the connection

5. If no Transfer-Encoding and invalid Content-Length, the recipient MUST treat that as an unrecoverable error, unless the field value is a comma-separated list, all list values are valid and are equal (obvious how to treat it then lol). Otherwise, the server MUST respond with 400 and close the connection. If it is a response received by a proxy, the proxy MUST close the connection to the server, discard message and send 502 to the client. If received by a client, it MUST close the connection and discard message.

6. If no Transfer-Encoding and valid Content-Length, Content-Length. If sender closes the connection or received times out, the recipient MUST consider the message as incomplete and close the connection

7. If it is a request, body length is zero

8. If it is a response, body length is read until server closes connection

A server SHOULD generate encoding or length delimited messages whenever possible

Request messages are never close-delimited.

A server MAY reject a request that contains a message body but not a Content-Length by responding with 411

Unless a transfer coding other than chunked has been applied, a client sending a request with a body SHOULD use a valid Content-Length if it is known in advance, rather than the chunked transfer coding

A user agent that sends a request with a body MUST either use Content-Length or chunked transfer coding. A client MUST NOT use the chunked coding unless it knows the server will handle HTTP/1.1 or later.

If the final response to the last request on a connection has been completely received and there remains additional data to read, a user agent MAY discard it, or attempt to determine if it belongs to a previous message body (e.g. Content-Length was incorrect). A client MUST NOT process, cache or forward such data as a separate response.

## 7. Transfer Codings

Transfer coding is a property of the message, not of the representation within

All transfer-coding names are case-insensitive. They are used in Transfer-Encoding and TE header fields.

### 7.1. Chunked Transfer Coding

```
chunked-body        ::= chunk*
                        last-chunk
                        trailer-section
                        CRLF

chunk               ::= chunk-size [ chunk-ext ] CRLF
                        chunk-data CRLF
chunk-size          ::= HEXDIG+
last-chunk          ::= "0"+ [ chunk-ext ] CRLF
chunk-data          ::= OCTET+

chunk-ext           ::= chunk-ext-single*
chunk-ext-single    ::= BWS ";" BWS chunk-ext-name
                        [ BWS "=" BWS chunk-ext-val ]
chunk-ext-name      ::= token
chunk-ext-val       ::= token / quoted-string

trailer-section     ::= (field-line CRLF)*
```

A recipient MUST be able to parse and decode the chunked transfer coding

Recipients MUST anticipate potentially large hexadecimal numerals and prevent parsing errors due to integer conversion overflows or precision loss due to integer representation

The chunked coding does not define any parameters. Their presence SHOULD be treated as an error

#### 7.1.1. Chunk Extensions

May be used for per-chunk metadata, like signature, hash, control information, whatever

A recipient MUST ignore unrecognized chunk extensions. A server out to reasonably limit the total length of received chunk extensions, and respond with a 4xx if that limit is exceeded

#### 7.1.2. Chunked Trailer Section

A recipient that removes the chunked coding MAY selectively retain or discard the received trailer fields. A recipient that retains MUST store/forward the trailer field separately from the received header fields, or merge the trailer field into the header section. A recipient MUST NOT merge a trailer field into the header section unless it is explicitly permitted and defined by the field

#### 7.1.3. Decoding Chunked

```
length := 0
while(read chunk-size, chunk-ext (if any), CRLF;
      chunk-size > 0) {
    read chunk-data, CRLF
    append chunk-data to content
    length += chunk-size
}
while(read trailer field;
      trailer field is not empty) {
    if(trailer fields are stored/forwarded separately) {
        append trailer field to existing trailer fields
    }
    else if(trailer field is understood and defined as mergeable) {
        merge trailer field with existing header fields
    }
    else {
        discard trailer field
    }
}
Content-Length := length
remove "chunked" from Transfer-Encoding
```

### 7.2. Transfer Codings for Compression

Following coding names are defined:

- compress (and x-compress) - RFC9110 Section 8.4.1.1.
- deflate - RFC9110 Section 8.4.1.2.
- gzip (and x-gzip) - RFC9110 Section 8.4.1.3.

These do not define any parameters. Parameters' presence SHOULD be treated as an error

### 7.3. Transfer Coding Registry

nobody cares

### 7.4. Negotiating Transfer Codings

The TE field is used in HTTP/1.1 to indicate what transfer codings (chunked implied) the cilent is willing to accept in the response, and whether it is willing to preserve trailer fields

A client MUST NOT send the chunked transfer coding in TE

```
TE: deflate
TE: 
TE: trailers, deflate;q=0.5
```

The client MAY rank the codings by preference using a case-insensitive "q" parameter. The rank value is a real number in range [0,1], where 0.001 is the least preferred, 1 is most preferred, 0 is unacceptable

If TE is empty or not present, only chunked (or none at all) is acceptable

"trailers" indicates that the sender will not discard trailer fields

A sender of TE MUST also send a "TE" connection option within the Connection header field

## 8. Handling Incomplete Messages

A server that receives an incomplete, for any reason, message, MAY send an error repsonse prior to closing the connection

A client that receives such a message MUST record the message as incomplete.

If a response terminated in the middle of the header section and the status code might rely on header fields, then the client cannot assume that meaning has been conveyed, and needs to repeat the request

A chunked encoded messagy body is incomplete if the zero-sized chunk hasn't been received. A message that uses a valid Content-Length is incomplete if less octets have been received. A response that uses neither is terminated by closure of the connection and, if the header section was received in tact, is considered complete unless a connection error had occurred

## 9. Connection Managemento

HTTP only presumes a reliable transport with in-order delivery of requests and corresponding in-order delivery of responses.

The "http" URI scheme indicates a default connection of TCP over IP, with a default TCP port of 80, but the client might be configured to use a proxy via some other connection, port or protocol.

### 9.1. Establishment

It is beyond the scope of this specification. Each HTTP connection maps to one underlying transport connection

### 9.2. Associating a Response to a Request

HTTP/1.1 relies on the order of response arrival to correspond exactly to the order in which requests are made on the same connection. More than 1 response per request only occurs when one or more informational responses precede a final response to the same request

A client MUST maintain a list of outstanding requests in order sent and MUST associate each received response message on the connection to the first outstanding request that has not yet received a final non-1xx response.

If a client receives a response without having outstanding requests, it MUST NOT consider that data to be a valid response. The client SHOULD close the connection, unless the data consists only of CRLF+

### 9.3. Persistence

HTTP implementations SHOULD support persistent connections

A recipient determines whether a connection is persistent based on the protocol version and Connectiom header of the last message, if any

- If the "close" connection option is present, the connection will not persist, else
- If protocol is HTTP/1.1+ will persist, else
- If HTTP/1.0, "keep-alive" connection option is present, the recipient is not a proxy or the message is a response, and the recipient wishes to oblige, the connection will persist, else
- The connection will close after the current response

A client that does not support persistent connections MUST send the "close" connection option in every request message

Same (MUST) applies to the servers in every non-1xx response

A client MAY send more requests on a persistent conneciton until it sends or receives "close" connection option or receives a HTTP/1.0 response without a "keep-alive" connection option

A server MUST either read the entire request message body or close the connection after sending its response. Client MUST do a similar thing with its requests

A proxy server MUST NOT maintain a persistent connection with an HTTP/1.0 client

#### 9.3.1. Retrying Requests

Implementations ought to anticipate the need to recover from asynchronous close events

#### 9.3.2. Pipelining

A client that supports persistent connnections MAY send multiple requests without waiting for each response, i.e. pipeline them. A server MAY process a sequence of such requests in parallel if they all have safe methods, but it MUST send the responses in corresponding order

A client that pipelines requests SHOULD retry unanswered requests if the connection closes before expected. When retrying after a non explicit connection closure a client MUST NOT pipeline immediately after connection establishment

A user agent SHOULD NOT pipeline requests after a non-idempotent method, until the final response status code for that method has been received, unless the user agent has means to handle the consequences

An intermediary that receives pipelines requests MAY pipeline those requests when forwarding them inbound. If the inbound connection fails before receiving a response, the pipelining intermediary MAY attempt to retry requests that have yet to receive a response if they all have idempotent methods. Otherwise it SHOULD forward any received responses and close the outbound connection(s)

### 9.4. Concurrency

A client ought to limit the number of simultaneous open connections to a given server

Nothing is mandated, but you better be responsible

### 9.5. Failures and Timeouts

A client or server that wishes to time out SHOULD issue a graceful close on the connection. Implementations SHOULD constantly monitor open connections for such signals and respond appropriately

A client, server or proxy MAY close the transport connection at any time. A server SHOULD sustain persistent connections when possible and allow the underlying transport's flow-control mechanisms to resolve overloads rather than terminate the connections.

A client sending a message body SHOULD monitor the network connection for an error response while it is transmitting the request. If a client ingers that the server does not wish to receive the message body and is closing the connection, the client SHOULD immediately stop transmitting and close the connection

### 9.6. Tear-down

The "close" connection option is defined as a signal that the sender will close the connection after the completion of the response. A sender SHOULD send a Connection header field when it intends to do this

With "close" a client indicates that this is the last request, and server - last response

Client sending "close" MUST NOT send further requests on that connection and MUST close the connection after reading the final response

A server that receives a "close" MUST initiate closure of the connection after sending the final response. The server SHOULD send a "close" in that final response. The server MUST NOT process any further requests on that connection

Server sending "close" MUST inititate closure after sending that response. The server MUST NOT process any requests on that connection after that

Client receiving a "close" MUST cease sending requests and close the connection after reading that response. Additional pipelined requests SHOULD NOT be assumed to be processed by the server

To avoid issues with TCP, typically server close writing first, and keeps reading until receiving a close from the client, or if the server is certain that the client has acknowledged everything. After that fully close

### 9.7. TLS Connection Initiation

[RFC8446]

HTTP client acts as the TLS client. All HTTP data MUST be sent as TLS "application data" but is otherwise treated like a normal connection for HTTP

### 9.8. TLS Connection Closure

[RFC8446]

When encountering an incomplete close, a client SHOULD treat as completed all requests for which it has received either

- everything speicifed by Content-Length or
- the terminal chunk for chunked Transfer-Encoding

Otherwise, the client SHOULD recover gracefully

Clients MUST send a closure alert before closing the connection. They MAY choose not to wait for the server's closure alert

Servers SHOULD be prepared to receive an incomplete close

Servers MUST attempt to initiate an exchange of closure alerts with the client before closing. Servers MAY close the connection after sending the closure alert

## 10. Enclosing Messages as Data

### 10.1. Media Type message/http

"message/http" media type can be used to enclose a single HTTP message, provided it obeys the MIME restrictions for all "message" types regarding line length and encodings. With this in mind, `obs-fold` may be used. A recipient MUST replace obs-fold with SP+

Details: read the spec

### 10.2. Media type application/http

"application/http" media type can be used to enclose a pipeline of one or more HTTP all requests or all response messages

Details: read the spec

## 11. Security Considerations

This section is about known security considerations regarding syntax and parsing. Other are covered in [RFC9110]

### 11.1. Response Splitting

Response splitting/CRLF injection - attacker sends encoded data within some parameter of the request that is later decoded and echoed in the response. If the decoded data looks like the end of a response and beginning of a new one, the content of the "second" response may be malicious. Attacker may then send another request, and pretend that the aforementioned response is related to this malicious request

Basically be careful with CRLFs

### 11.2. Request Smuggling

guess

### 11.3. Message Integrity

HTTP does not check for message integrity itself.

### 11.4. Message Confidentiality

Again, not the concern of HTTP itself

## 12. IANA Considerations

don't care

### 12.1. Field Name Registration

Close, MIME-Version, Transfer-Encoding

### 12.2. Media Type Registration

don't care

### 12.3. Transfer Coding Registration

read original (actually useful tho)

### 12.4. ALPN Protocol ID Registration

don't care

## 13+

don't care
