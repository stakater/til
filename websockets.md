# WebScocket

## What is WebSocket protocol?

WebSockets is a bi-directional, full-duplex, persistent connection between a web browser and a server. Once a WebSocket connection is established the connection stays open until the client or server decides to close this connection.

## A Deeper Look into the SOP and CORS Standard

By default, modern browsers adhere to the SOP which is a security mechanism that places limitations on how the requesting 
origin can interact with resources retrieved from another domain if the origins differ. To limit the security ramifications 
of cross-origin requests, browsers restrict access to a cross-origin response in accordance with the SOP.

CORS is a mechanism that enables browsers to retrieve then grant access to resources via requests originating from a 
different domain than that of what is currently being browsed. Developers utilize CORS to relax or disable the SOP 
entirely to allow front-end applications to access cross-origin resources.

Cross-origin HTTP requests occur when a client issues a request from an entirely different domain, port, or using a 
different protocol than the domain currently browsed. An example of a cross-origin request would be if a request 
from https://example.com is issued to http://example.com:3000.This would be a cross-origin request due to the different 
protocols (HTTPS ->HTTP) and ports (443 -> 3000).

CORS policies inform the browser which domains are allowed to access the response object of a request, if cookies should 
be sent within requests, and which HTTP methods are allowed via HTTP response headers (just to mention a few restrictions 
that could be imposed by CORS policies). The cross-origin resource standard includes several HTTP headers; however, I will 
mainly focus on the Access-Control-Allow-Origin header as it permits browsers to grant cross-origin access to response objects.

As mentioned, the Access-Control-Allow-Origin header informs the browser whether or not to grant the requesting domain 
access to the response object. The value of this header specifies the origin(s) that is permitted to access the response data.
Examples of this will be:

Access-Control-Allow-Origin: http://example.com

and

Access-Control-Allow-Origin: *

where the first header informs the browser to limit resource access to http://example.com, and the second informs the browser 
to grant resource access to all domains. Let us take a look at a cross-site request and response to paint a clearer picture.

GET /info.png HTTP/1.1
Host: http://example.com
Referer: http://evil.com
Origin: http://evil.com
The request headers above shows an HTTP GET request originating from http://evil.com to retrieve info.png from 
http://example.com.

HTTP/1.1 200 OK
Connection: close
Access-Control-Allow-Origin: http://example.com

The response to the cross-site request (above) informs the browser that info.png should only be accessible to the
http://example.comdomain. The browser will issue the GET request and receive the content; however, it will not grant access 
to the response object to http://evil.com due to the origin restriction set by the Access-Control-Allow-Origin header.

It should be understood that CORS enables cross-origin access to response objects. Considering that, even without the 
presence of the custom CORS policy, browsers will prevent access to response objects of cross-origin requests, so access 
to the response will not be granted.

## What is Socket.io?

Socket.io framework includes a client API that makes developing WS clients straightforward.

## Key Takeaways

- Browsers prevent access to response objects when a cross-origin request is made by default.
- CORS policies enable browsers to grant access to response data when a cross-origin request is made.
- SOP and CORS apply only to the HTTP URI scheme.
- Client-server WebSocket handshakes occur over the HTTP protocol.
- Attackers could maliciously connect to WS servers without authentication/authorization.
- Restrict access to the WebSocket server via origin validation.
- Validating the origin is not a replacement for authentication or authorization verification.

## Spring & WebSockets

WebSocket is an exciting way to send messages between applications, especially when one of those applications is running 
within a web browser. It’s critical for writing highly interactive web applications that seamlessly transfer data to and 
from the server.

Spring’s WebSocket support includes a low-level API that lets you work with raw WebSocket connections. Unfortunately, 
WebSocket support is not ubiquitous among web browsers, servers, and proxies. Therefore, Spring also supports SockJS, 
a protocol that falls back to alternative communication schemes when WebSocket doesn’t work.

Spring also offers a higher-level programming model for handling WebSocket mes- sages using the STOMP wire-level protocol. 

In this higher-level model, STOMP messages are handled in Spring MVC controllers, similarly to how HTTP messages are handled.

## WebSocket Protocol

WebSocket is a protocol which communicates between browser and web server. WebSocket works over TCP protocol. It opens a 
socket connection between browser and web server and start communication. WebSocket embeds sub protocol easily. In this 
application we are using STOMP over WebSocket. 

## SockJS
SockJS is a java script library which provides websocket like object for browsers. SockJS provides cross browser 
compatibility and supports STOMP protocol to communicate with any message broker. SockJS works in the way that we need to 
provide URL to connect with message broker and then get the stomp client to communicate. 

### Why SockJS?

SockJS lets applications use a WebSocket API but falls back to non-WebSocket alternatives when necessary at runtime, without the need to change application code.

## STOMP Protocol
STOMP is Streaming Text Oriented Messaging Protocol. A STOMP client communicates to a message broker which supports STOMP 
protocol. STOMP uses different commands like connect, send, subscribe, disconnect etc to communicate. 

## Tools

- SockJS
- STOMP
- Spring Boot
- Apache Kafka

## References

- Chapter 18 - Messaging with WebSocket and STOMP from Spring in Action 4th Edition
- https://www.youtube.com/watch?v=nxakp15CACY
- [WebSockets not Bound by SOP and CORS?](https://blog.securityevaluators.com/websockets-not-bound-by-cors-does-this-mean-2e7819374acc)
- https://www.baeldung.com/websockets-spring
