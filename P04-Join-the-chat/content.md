---
title: Give users an identity
slug: Join-the-chat
---

## Zen and the Art of Socket Maintenance

Before we move on to our next feature, let's make a separate file for our socket listeners. This will reduce the socket clutter in our app.js

```bash
$ mkdir sockets
$ cd sockets
$ touch chat.js
$ cd ..
```

Update app.js to read the new file.

```javascript
//app.js
const io = require('socket.io')(server);
io.on("connection", (socket) => {
  // This file will be read on new socket connections
  require('./sockets/chat.js')(io, socket);
})
```

Your chat.js should look like this

```javascript
//chat.js
module.exports = (io, socket) => {
  //Future socket listeners will be here
}
```

## What's in a name?
Let's update our handlebars to have a username form.
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Make Chat</title>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <script src="socket.io/socket.io.js"></script>
    <script src="/public/index.js"></script>
    <link href='https://cdnjs.cloudflare.com/ajax/libs/flexboxgrid/6.3.1/flexboxgrid.min.css'></link>
    <link href='/public/index.css' rel='stylesheet' type='text/css'></link>
  </head>
  <body>

    <form class="usernameForm">
      <input id="usernameInput" placeholder="Username"></input>
      <button id="createUserBtn">Join Chat</button>
    </form>

  </body>
</html>
```

I'll be a pal and give you some CSS to use too.
```bash
$ cd public
$ touch index.css
$ cd ..
```

```css
/* index.css */
*{
  font-family: helvetica;
}

.usernameForm{
  display: flex;
  width : 50%;
  margin-left: auto;
  margin-right: auto;
  padding: 10px;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

#usernameInput{
  font-size: 20px;
  padding: 5px;
  margin-bottom: 5px;
  width : 100%:
}

#createUserBtn{
  width : 45%;
  font-size: 20px;
  font-weight: bold;
  padding: 5px;
}
```

Feel free to reload the page to make sure your username form looks good to go.
Let's add some client script to send the new username to the server.

```javascript
// index.js
$(document).ready(()=>{
  const socket = io.connect();

  $('#createUserBtn').click((e)=>{
    e.preventDefault();
    let username = $('#usernameInput').val();
    if(username.length > 0){
      //Emit to the server the new user
      socket.emit('new user', username);
      $('.usernameForm').remove();
    }
  });

})
```

This code **emits** to the server the new username.
Now if you attempt to join the chat you will see that absolutely nothing happens.

Why is that? We still have to add the correct socket listener on the backend.

Update your chat.js accordingly
```javascript
// chat.js
module.exports = (io, socket) => {

  // Listen for "new user" socket emits
  socket.on('new user', (username) => {
    console.log(`${username} has joined the chat! ✋`);
  })

}
```

Now whenever the client **emits** a *"new user"* request, our server will be **on** it.

Go ahead and make a username, then check the server logs.

## What goes around, comes around
We have successfully sent data from the client to the server with sockets.

Now let's send data from the server to all clients.

The server can **emit** as well.

```javascript
// chat.js
socket.on('new user', (username) => {
  console.log(`✋ ${username} has joined the chat! ✋`);
  //Send the username to all clients currently connected
  io.emit("new user", username);
})
```

Notice how the server says *io.emit* instead of *socket.emit*.

**io.emit** sends data to all clients

**socket.emit** sends data to the client that sent the original data to the server

There's some more ways to emit data that we'll go in to later. These two are the most common.

Now update your client script to be **on** (listening) for the *"new user"* **emit**.

```javascript
// index.js
$(document).ready(() => {

  const socket = io.connect();

  $('#createUserBtn').click((e) => {
    e.preventDefault();
    let username = $('#usernameInput').val();
    if(username.length > 0){
      //Emit to the server the new user
      socket.emit('new user', username);
      $('.usernameForm').remove();
    }
  });

  //socket listeners
  socket.on('new user', (username) => {
    console.log(`✋ ${username} has joined the chat! ✋`);
  })

})
```

Now both your server and clients will be logging out the new users.

Open up two instances of http://localhost:3000/ and check their logs.

## Wow, that's cool!
That's right bud, and this next part of the tutorial will be even cooler.
