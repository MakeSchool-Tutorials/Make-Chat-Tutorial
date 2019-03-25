---
title: "Give Users an Identity"
slug: join-the-chat
---

1. ~~Build out a basic view~~
1. ~~Integrate sockets~~
1. **Implement user form**
    1. **Pass the socket server and the socket itself**
    1. **Build out the structure of the user form**
    1. **Send info from the client to the server using the user form**
    1. **Send info from the server to all clients**
1. Style and send messages
1. Connect/disconnect users
1. Create/persist/join channels

## Zen and the Art of Socket Maintenance

Before we move on to our next feature, let's make a separate file for our socket listeners. This will reduce the socket clutter in our `app.js`

>[action]
> make a `/sockets` folder with a `chat` file in it
>
```bash
$ mkdir sockets
$ touch sockets/chat.js
```

Update `app.js` to require the new file. We're going to pass the socket server (`io`) and the socket itself (`socket`) into our file.

>[action]
> Update the body of `io.on(...)` in `app.js` to the following:
>
```javascript
//app.js
const io = require('socket.io')(server);
io.on("connection", (socket) => {
  // This file will be read on new socket connections
  require('./sockets/chat.js')(io, socket);
})
```
>
> Update `/sockets/chat.js` to the following:
>
```javascript
//chat.js
module.exports = (io, socket) => {
  //Future socket listeners will be here
}
```

# What's in a name?

Let's update our view to have a username form.

>[action]
> Update `/views/index.handlebars` to the following:
>
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
>
    <form class="usernameForm">
      <input id="usernameInput" placeholder="Username"></input>
      <button id="createUserBtn">Join Chat</button>
    </form>
>
  </body>
</html>
```

I'll be a pal and give you some CSS to use too. Remember to make your `public/index.css` first and require it in your `<head></head>` tag.

>[action]
> Create `/public/index.css`
>
```bash
$ touch public/index.css
```
>
> Paste the following code into `/public/index.css`:
>
```css
/* public/index.css */
*{
  font-family: helvetica;
}
>
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
>
#usernameInput{
  font-size: 20px;
  padding: 5px;
  margin-bottom: 5px;
  width : 100%;
}
>
#createUserBtn{
  width : 45%;
  font-size: 20px;
  font-weight: bold;
  padding: 5px;
}
```

Reload the page to make sure your username form looks good to go.

# Submitting the Form Via WebSocket

Let's add some client script to send the new username to the server. Notice that we are not going to use an HTTP request, instead we can use a *socket event* on the connection we already have created! We'll create a new event called `new user`. On the server we'll later create a listener to listen for events named this.

>[action]
> Update `/public/index.js` to the following:
>
```javascript
// index.js
$(document).ready(()=>{
  const socket = io.connect();
>
  $('#createUserBtn').click((e)=>{
    e.preventDefault();
    let username = $('#usernameInput').val();
    if(username.length > 0){
      //Emit to the server the new user
      socket.emit('new user', username);
      $('.usernameForm').remove();
    }
  });
>
})
```

This code **emits** to the server the new username. Emit allows for registering custom events.

Now if you attempt to join the chat you will see that absolutely nothing happens. Blank page.

Why is that? We still have to add a *matching socket listener* on the backend that listens for an event called `new user`.

>[action]
> Update `/sockets/chat.js` to the following:
>
```javascript
// chat.js
module.exports = (io, socket) => {
>
  // Listen for "new user" socket emits
  socket.on('new user', (username) => {
    console.log(`${username} has joined the chat! ✋`);
  })
>
}
```

Now whenever the client **emits** a *"new user"* request, our server will be **on** it.

Go ahead and make a username, then check the server logs.

# What Goes Around, Comes Around

We have successfully sent data from the client to the server with sockets.

Now let's send data from the server to all clients. The server can **emit** as well.

>[action]
> Update the body of `socket.on(...)` in `/sockets/chat.js` to the following:
>
```javascript
// chat.js
socket.on('new user', (username) => {
  console.log(`✋ ${username} has joined the chat! ✋`);
  //Send the username to all clients currently connected
  io.emit("new user", username);
})
```

Notice how the server says *io.emit* instead of *socket.emit*.

**io.emit** sends data to all clients on the connection.

**socket.emit** sends data to the client that sent the original data to the server.

There's some more ways to emit data that we'll go in to later. These two are the most common.

Now we have to setup the client to listen for any `new user` events coming from the server. we'll use the same sort of lingo: **on** (listening) *"new user"*.

>[action]
> Update `/public/index.js` to the following:
>
```javascript
// index.js
$(document).ready(() => {
>
  const socket = io.connect();
>
  $('#createUserBtn').click((e) => {
    e.preventDefault();
    let username = $('#usernameInput').val();
    if(username.length > 0){
      //Emit to the server the new user
      socket.emit('new user', username);
      $('.usernameForm').remove();
    }
  });
>
  //socket listeners
  socket.on('new user', (username) => {
    console.log(`✋ ${username} has joined the chat! ✋`);
  })
>
})
```

Now both your server and clients will be logging in the new users.

To simulate two connections, open up two browser windows and direct them both to [http://localhost:3000/](http://localhost:3000/), create different users on each, and check both browser window's JavaScript consoles.

# Now Commit

```bash
$ git add .
$ git commit -m 'Users can log in'
$ git push
```

## Wow, that's cool!

That's right bud, and this next part of the tutorial will be even cooler.
