---
title: "What we're building"
slug: start-slacking
---

If you've been attending Make School for any amount of time now, you should be very acquainted with the chat program Slack.

This live messaging app, along with its little brother Discord, rely heavily on realtime communication.

In this tutorial we're going to be building our very own live messaging application.

# What You'll Need to Know

To begin this tutorial you should already have familiarity with the following domains and tools:

* Client-side JavaScript
* jQuery
* Node.js & Express.js

To gain familiarity with these topics, you can complete the following tutorials and e-courses:

1. [Code Academy JavaScript](https://www.codecademy.com/learn/introduction-to-javascript)
1. [Code Academy jQuery](https://www.codecademy.com/learn/learn-jquery)
1. [Giphy Search Tutorial](https://www.makeschool.com/academy/track/giphy-search-app-with-node-js)
1. [Rotten Potatoes Tutorial](https://www.makeschool.com/academy/track/rotten-potatoes---movie-reviews-with-express-js)

# HTML5 Bidirectional Communication => WebSocket

Before we jump into building with websockets, lets take a second to look at where the WebSocket standard came from and it works. Much of this complexity is buried into the libraries and tools we use, so lets take a minute to look at them.

As HTML5 was being developed, it became clear that the web needed a bidirectional **full-duplex** standard to allow for bidirectional communication. A new standard called WebSocket was recommended in June of 2008 by Michael Carter—an influential HTML5 game developer.

In February 2010, Google (being a champion of HTML5) made Chrome 4 the first browser to ship a full support of the standard and enabled by default with Safari 5.0.0 in a close second. The last was Internet Explorer 10 in December 2011.

The WebSocket standard begins with an HTTP handshake, but then switches to the WebSocket standard that does not conform to the HTTP protocol.

![WebSocket Diagram](WebSockets-Diagram.png)

Here's an example of how the request for the handshake and the server's response looks:

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```

Server response:
```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```

The request sends a `Sec-WebSocket-Key` which contains base64-encoded random bytes, and response responds with with a hash of the key in the `Sec-WebSocket-Accept` attribute which prevents resending old messages. This key/hash pattern does not provide any authentication, privacy, or integrity. WebSocket has unique security and privacy concerns.

For more information read this [WebSocket documentation](https://hpbn.co/websocket/).

# What We are Going to Build

### User Stories

1. Upon entering the application, I can choose a username to go by.
2. I can enter a message that shows up on all clients.
3. I can join / create chat channels.
4. I will never have to refresh the page to receive new messages.
5. I can start a private message with another user. (Stretch Challenge)

### Wireframe

Let's look at a simple wireframe of what we want to build. Basically it is a chat application with very little underutilized space.

![Wireframe](make-chat-wireframe.png)

# This sounds impossible...

And it certainly would be next to impossible... if it weren't for a little library called socket.io!

[Socket.io](socket.io) is a lightweight wrapper you can use to implement websockets using the WebSocket standard.

>[info]
> Remember that Socket.io has both a **server** and a **client** side to handle both the sending and receiving of websocket messages.

Some types of applications that use socket.io are the following:

1. Live Messaging / Chat Rooms
2. Realtime Data Analytics
3. Multiplayer Games such as Agar.io or Slither.io
4. Applications you make after this tutorial (hopefully)
