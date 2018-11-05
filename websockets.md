# WebScocket

## What is WebSocket protocol?

WebSockets is a bi-directional, full-duplex, persistent connection between a web browser and a server. Once a WebSocket connection is established the connection stays open until the client or server decides to close this connection.

WebSocket is a full-duplex communications protocol layered over TCP. It is typically used for interactive communication between a user’s browser and a back-end server. An example would be a chat server with real-time communications between the server and the connected clients. Another example would be a Stock Trading application where the server sends stock price variations to subscribed clients without an explicit client request.

A WebSocket is a communication channel which uses TCP as the underlying protocol. It is initiated by the client sending a HTTP request to the server requesting a connection upgrade to WebSocket. If the server supports WebSockets, the client request is granted and a WebSocket connection is established between the two parties. After this connection is established, all communication happens over the WebSocket and the HTTP protocol is no longer used.

## STOMP over WebSocket

The WebSocket communication protocol itself does not mandate any particular messaging format. It is up to the applications to agree upon the format of the messages exchanged. This format is referred to as the subprotocol. (Kinda similar to how the web browser and web server have agreed to using the HTTP protocol over TCP sockets.)

One commonly used format is the STOMP protocol (Streaming Text Oriented Message Protocol) used for general purpose messaging. Various message oriented middle-ware (MOM) systems such as Apache ActiveMQ, HornetQ and RabbitMQ support STOMP.

The Spring Framework implements WebSocket communication with STOMP as the messaging protocol.

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

SockJS, the best and the most comprehensive WebSocket browser fallback options. You will need fallback options in browsers that don't support WebSocket and in situations where network proxies prevent its use. Simply put SockJS enables you to build WebSocket applications

## STOMP Protocol

STOMP is Streaming Text Oriented Messaging Protocol. A STOMP client communicates to a message broker which supports STOMP 
protocol. STOMP uses different commands like connect, send, subscribe, disconnect etc to communicate. 

STOMP is a messaging protocol created with simplicity in mind. It is based on frames modelled on HTTP. A frame consists of a command, optional headers, and optional body.

## Technologies

- Java
- Maven
- SockJS
- STOMP
- Spring Boot
- Apache Kafka
- WebSockets

## AWS ALB

- It supports websockets natively
- [Sticky Sessions](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html#sticky-sessions) : Sticky sessions are a mechanism to route requests to the same target in a target group. This is useful for servers that maintain state information in order to provide a continuous experience to clients. To use sticky sessions, the clients must support cookies.
- We must have sessionAfinity on the kubernetes service
```
apiVersion: v1
kind: Service
metadata:
  ...
spec:
  type: NodePort
  sessionAffinity: ClientIP
```  

## Why need RabbitMQ?

We have websockets implemented over the STOMP protocol. These websockets were currently maintained by our application container. Since we wanted to achieve a multi-instance production environment we would have to use a single message broker so messaging can reach any connected endpoint, not only the ones known by each application container.

Our application used websockets to send notifications to connected users. So far, so good. The thing is that we were using our application container message broker implementation and when running multiple application containers, notifications could obviously not reach all intended users because their connection might have been established with any application container, possibly not the with the one sending the message.

RabbitMQ is a message broker software that can be integrated to the Spring ecosystem. In this scenario our application container receives websockets connections and relay them to RabbitMQ. This way all websockets are knowledgeable by any application container and can get notified.

RabbitMQ is a message broker solution which supports multiple messaging protocols. STOMP is one among these supported protocols.

Spring WebSocket is the Spring module that enables WebSocket-style messaging support. As Spring WebSocket’s documentation states, the WebSocket protocol defines an important new capability for web applications: full-duplex, two-way communication between client and server.

Spring WebSocket makes it straightforward to enable websockets and work as a relay to a message broker such as RabbitMQ. This way you may run multiple application container connected to the same instance or maybe a cluster of RabbitMQ instances.

The application is clustered ( many nodes behind a Load Balancer) so it requires more complicate solution than just a simple java server which can send and receive websocket frames. Because of that, we decided to use a STOMP Broker which will allow us to scale for multiple nodes.

There are few Brokers which implements the STOMP protocol, we have chosen RabbitMQ since it has larger community and it seems more reliable(it also has nice administration UI).

Extracted [from](http://djeison.me/2017/11/04/spring-websocket-rabbitmq/)

## References

- Chapter 18 - Messaging with WebSocket and STOMP from Spring in Action 4th Edition
- https://www.youtube.com/watch?v=nxakp15CACY
- Run and go through this sample app: https://github.com/jorgeacetozi/ebook-chat-app-spring-websocket-cassandra-redis-rabbitmq
- Run and go through this sample app: https://github.com/salmar/spring-websocket-chat
- [WebSockets not Bound by SOP and CORS?](https://blog.securityevaluators.com/websockets-not-bound-by-cors-does-this-mean-2e7819374acc)
- https://www.baeldung.com/websockets-spring

## Reference Implementations

- https://github.com/rstoyanchev/spring-websocket-portfolio

## Diagrams

### Application & Message Broker Solution

One approach is to make the application handle incoming messages and serve as intermediary between web clients and the message broker. Messages from clients can flow to the broker through the application and reversely messages from the broker can flow back to clients through the application. This gives the application a chance to examine the incoming message type and "destination" header and decide whether to handle the message or pass it on to the broker.

![Application and Message-Broker Solution](/diagrams/m2-app-server-broker.png)

### Message-Broker Solution

One server-side option is a pure message-broker solution where messages are sent directly to a traditional message broker like RabbitMQ, ActiveMQ, etc. Most, if not all brokers, support STOMP over TCP but increasingly they support it over WebSocket too while RabbitMQ goes further and also supports SockJS. Our architecture would look like this:

![Application and Message-Broker Solution](/diagrams/me-broker-solution.png)

This is a robust and scalable solution but arguably not the best fit for the problem at hand. Message brokers have typically been used within the enterprise. Exposing them directly over the web isn't ideal.

If we've learned anything from REST it is that we don't want to expose details about the internals of our system like the database or the domain model.

Furthermore, as a Java developer you want to apply security, validation, and add application logic. In a message-broker solution the application server sits behind the message broker, which is a significant departure from what most web application developer are used to.

This is why a library such as socket.io is popular. It is simple and it targets the needs of web applications. On other hand we must not ignore the capabilities of message brokers to handle messages, they are really good at it and messaging is a hard problem. We need the best of both.

Extracted [from](http://assets.spring.io/wp/WebSocketBlogPost.html)
