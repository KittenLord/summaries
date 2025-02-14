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
