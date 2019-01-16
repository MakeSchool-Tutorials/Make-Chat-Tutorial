---
title: "Creating Channels"
slug: change-the-channel
---

Wow you made it this far. I hope you've been taking breaks because this next part is a bit much.

So currently all messages are in one chat room, but if we want to be like Slack or Discord we need there to be many channels. We're going to use Socket.io's built in `rooms` functionality to implement these channels.

We're now going to create different "channels" for users to create or join. Let's add some functionality to the channel creator on the client. Each channel will be a sub-chatroom that is logged as belonging to a parent socket connection.

# Channel Your Energy

So we are going to make a form with a `#newChannelBtn` and make it generate a new channel.

```js
//index.js
$('#newChannelBtn').click( () => {
  let newChannel = $('#newChannelInput').val();

  if(newChannel.length > 0){
    // Emit the new channel to the server
    socket.emit('new channel', newChannel);
    $('#newChannelInput').val("");
  }
})
```

Over on the server, make sure the event is registered:

```js
// chat.js

  socket.on('new channel', (newChannel) => {
    console.log(newChannel);
  })
```

Test that that works before going forward. Baby steps, baby steps.

# Persisting Channels

Just like the users, we're going to want to save each channel locally so that they persist across new clients.

```javascript
//app.js
const io = require('socket.io')(server);
let onlineUsers = {};
//Save the channels in this object.
let channels = {"General" : []}

io.on("connection", (socket) => {
  // Make sure to send the channels to our chat file
  require('./sockets/chat.js')(io, socket, onlineUsers, channels);
})
```

Notice how we have the **General** channel already included in our object. We want this channel to be available without having to be created.

The array value that comes with the channel key will be used to save each channel's messages.

Let's now have our server be `on()` `"New Channel"`.

```javascript
//chat.js

//Make sure to add channels to module.exports parameters
module.exports = (io, socket, onlineUsers, channels) => {

  ...

  socket.on('new channel', (newChannel) => {
    //Save the new channel to our channels object. The array will hold the messages.
    channels[newChannel] = [];
    //Have the socket join the new channel room.
    socket.join(newChannel);
    //Inform all clients of the new channel.
    io.emit('new channel', newChannel);
    //Emit to the client that made the new channel, to change their channel to the one they made.
    socket.emit('user changed channel', {
      channel : newChannel,
      messages : channels[newChannel]
    });
  })
}
```

## Go To Your Room!

Socket.io has this fancy thing called `rooms` in which different sockets(clients) can `.join()`.

In the code we just put in, we did `socket.join(newChannel)` which is telling the socket to join the new channel room.

Rooms are great, because you can emit only to that room. You will see what I mean very soon.

Let's update the client to display all channels that exist and mark for each user which channel they are on (which Socket.io `room` they are in).

```javascript
//index.js

// Add the new channel to the channels list (Fires for all clients)
socket.on('new channel', (newChannel) => {
  $('.channels').append(`<div class="channel">${newChannel}</div>`);
});

// Make the channel joined the current channel. Then load the messages.
// This only fires for the client who made the channel.
socket.on('user changed channel', (data) => {
  $('.channel-current').addClass('channel');
  $('.channel-current').removeClass('channel-current');
  $(`.channel:contains('${data.channel}')`).addClass('channel-current');
  $('.channel-current').removeClass('channel');
  $('.message').remove();
  data.messages.forEach((message) => {
    $('.messageContainer').append(`
      <div class="message">
        <p class="messageUser">${message.sender}: </p>
        <p class="messageText">${message.message}</p>
      </div>
    `);
  });
})
```

Fantastic! Go ahead and try making a new channel. You should see that you join it automatically, and that the new channel is created for all clients.

# That's not supposed to go there!

Unfortunately, messages are currently being broadcasted to every client, regardless of which channel they came from. Let's fix that!

Let's go to the client message creation function.

```javascript
//index.js
$('#sendChatBtn').click((e) => {
  e.preventDefault();
  // Get the client's channel
  let channel = $('.channel-current').text();
  let message = $('#chatInput').val();
  if(message.length > 0){
    socket.emit('new message', {
      sender : currentUser,
      message : message,
      //Send the channel over to the server
      channel : channel
    });
    $('#chatInput').val("");
  }
});
```

We should now update the `"new message"` listener on the server to pay attention to which user and channel the message was sent on.

```javascript
//chat.js
socket.on('new message', (data) => {
  //Save the new message to the channel.
  channels[data.channel].push({sender : data.sender, message : data.message});
  //Emit only to sockets that are in that channel room.
  io.to(data.channel).emit('new message', data);
});
```

And finally let's update the `"new message"` listener on the client to respect channel.

```javascript
//index.js
socket.on('new message', (data) => {
  //Only append the message if the user is currently in that channel
  let currentChannel = $('.channel-current').text();
  if(currentChannel == data.channel){
    $('.messageContainer').append(`
      <div class="message">
        <p class="messageUser">${data.sender}: </p>
        <p class="messageText">${data.message}</p>
      </div>
    `);
  }
})
```

Great, now the messages will only go to their corresponding channels.

# Changing Channels

Let's let users change their channel by clicking on it.

```javascript
//index.js
const socket = io.connect();
let currentUser;
socket.emit('get online users');
//Each user should be in the general channel by default.
socket.emit('user changed channel', "General");

//Users can change the channel by clicking on its name.
$(document).on('click', '.channel', (e)=>{
  let newChannel = e.target.textContent;
  socket.emit('user changed channel', newChannel);
});
```

Let's add a server socket listener for `"user changed channel"`

```javascript
//chat.js

//Have the socket join the room of the channel
socket.on('user changed channel', (newChannel) => {
  socket.join(newChannel);
  socket.emit('user changed channel', {
    channel : newChannel,
    messages : channels[newChannel]
  });
})
```

Check out your application. Everything should now be working as it should!

# What now??

Well I have to say, I'm very proud of you for completing this tutorial!

Some future enhancements you could do

1. Implementing a private message system.
1. Using Socket.io's "channels" for multiple make chat teams to exist
1. Using Socket.io's "rooms" to make themed chat rooms inside a channel, like "general" and "random"
1. Dropping the client into an Electron app to make a desktop app.
1. Adding a Team and a User resource and persisting teams and authenticating users.
1. Great article on Socket.io's strengths and weaknesses: https://dzone.com/articles/socketio-the-good-the-bad-and-the-ugly

You have all the tools at your disposal to add these enhancements, I believe in you!