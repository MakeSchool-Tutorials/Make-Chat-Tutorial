---
title: "Creating Channels"
slug: change-the-channel
---

1. ~~Build out a basic view~~
1. ~~Integrate sockets~~
1. ~~Implement user form~~
1. ~~Style and send messages~~
1. ~~Connect/disconnect users~~
1. **Create/persist/join channels**
    1. **Create a button to generate a new channel**
    1. **Persist channels locally**
    1. **Join other channels**
    1. **Ensure messages go to their designated channels**
    1. **Allow users to change channels**

So currently all messages are in one chat room, but if we want to be like Slack or Discord we need there to be many channels. We're going to use Socket.io's built in [rooms](https://socket.io/docs/rooms-and-namespaces/#Rooms) functionality to implement these channels.

We're now going to create different "channels" for users to create or join. Let's add some functionality to the channel creator on the client. Each channel will be a sub-chatroom that is logged as belonging to a parent socket connection.

# Channel Your Energy

So we are going to make a form with a `#newChannelBtn` and make it generate a new channel.

>[action]
> Create a `newChannelBtn` handler near your other button handlers in `/public/index.js`:
>
```js
...
>
$('#newChannelBtn').click( () => {
  let newChannel = $('#newChannelInput').val();
>
  if(newChannel.length > 0){
    // Emit the new channel to the server
    socket.emit('new channel', newChannel);
    $('#newChannelInput').val("");
  }
})
```

Over on the server, make sure the event is registered:

>[action]
> Register the event in `/sockets/chat.js`:
>
```js
...
>
  socket.on('new channel', (newChannel) => {
    console.log(newChannel);
  })
```

Test that your `newChannel` console log is showing before going forward. Baby steps, baby steps.

# Now Commit

```bash
$ git add .
$ git commit -m 'Initial new channel'
$ git push
```

# Persisting Channels

Just like the users, we're going to want to save each channel locally so that they persist across new clients.

>[action]
> add `channels` to your `socket.io` code in `app.js`:
>
```javascript
//app.js
...
>
const io = require('socket.io')(server);
let onlineUsers = {};
//Save the channels in this object.
let channels = {"General" : []}
>
io.on("connection", (socket) => {
  // Make sure to send the channels to our chat file
  require('./sockets/chat.js')(io, socket, onlineUsers, channels);
})
...
```

Notice how we have the **General** channel already included in our object. We want this channel to be available without having to be created.

The array value that comes with the channel key will be used to save each channel's messages.

Now let's have our server be `on()` for `"New Channel"`.

>[action]
> Update `/sockets/chat.js` to be on for `new channel`. Remember to update the `module.exports` line to include `channels`:
>
```javascript
//chat.js
>
//Make sure to add channels to module.exports parameters
module.exports = (io, socket, onlineUsers, channels) => {
>
  ...
>
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

# Go To Your Room!

Socket.io has this fancy thing called `rooms` in which different `sockets(clients)` can `.join()`.

In the code we just put in, we did `socket.join(newChannel)` which is telling the socket to join the new channel room.

Rooms are great, because you can emit only to that room. You will see how that works very soon.

Let's update the client to display all channels that exist and mark for each user which channel they are on (which Socket.io `room` they are in).

>[action]
> Update `/public/index.js` to be `on` for `new channel` and `user changed channel`:
>
```javascript
...
>
// Add the new channel to the channels list (Fires for all clients)
socket.on('new channel', (newChannel) => {
  $('.channels').append(`<div class="channel">${newChannel}</div>`);
});
>
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

# Now Commit

```bash
$ git add .
$ git commit -m 'Users can create new channels and join'
$ git push
```

# That's not supposed to go there!

Unfortunately, messages are currently being broadcasted to every client, regardless of which channel they came from. Try it out yourself to see.

Let's fix that!

Let's go to the client message creation function and make sure it only goes to the right channel.

>[action]
> Update `/public/index.js` the `sendChatBtn` click handler to involve channels:
>
```javascript
...
>
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

We should now update the `new message` listener on the server to pay attention to which user and channel the message was sent on.

>[action]
> Update the `new message` listener in `/sockets/chat.js` to pay attention to channels:
>
```javascript
...
>
socket.on('new message', (data) => {
  //Save the new message to the channel.
  channels[data.channel].push({sender : data.sender, message : data.message});
  //Emit only to sockets that are in that channel room.
  io.to(data.channel).emit('new message', data);
});
```

And finally let's update the `new message` listener on the client to respect channel.

>[action]
> Update the `new message` listener in `/public/index.js` to pay attention to channels:
>
```javascript
...
>
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

# Now Commit

```bash
$ git add .
$ git commit -m 'Messages go to correct channel'
$ git push
```

Great, now the messages will only go to their corresponding channels. However, if you try sending a message, you won't see it!

That's cause we haven't specified a default channel (General) for a user to join. Let's fix that now.

# Changing Channels

Let's let users change their channel by clicking on it.

>[action]
> Update the top of `/public/index.js` to have an `emit` to `user changed channel`, and a click handler to allow for them to change a channel by clicking on it:
>
```javascript
//index.js
$(document).ready(() => {
>
  const socket = io.connect();
  let currentUser;
  socket.emit('get online users');
  //Each user should be in the general channel by default.
  socket.emit('user changed channel', "General");
>
  //Users can change the channel by clicking on its name.
  $(document).on('click', '.channel', (e)=>{
    let newChannel = e.target.textContent;
    socket.emit('user changed channel', newChannel);
  });
>
...
>
});
...
```

Finally, Let's add a server socket listener for `user changed channel`

>[action]
> Include a server socket listener for `user changed channel` in `/sockets/chat.js`:
>
```javascript
...
>
//Have the socket join the room of the channel
socket.on('user changed channel', (newChannel) => {
  socket.join(newChannel);
  socket.emit('user changed channel', {
    channel : newChannel,
    messages : channels[newChannel]
  });
>
...
>
})
```

Check out your application. Everything should now be working as it should! Great work!

As you go through this BEW course, see if you can relate what you learned in this tutorial to the lessons in class!

# Now Commit

```bash
$ git add .
$ git commit -m 'Implemented changing channels'
$ git push
```

# Stretch Challenges

Some future enhancements you could do

>[challenge]
>
1. Implementing a private message system.
1. Using Socket.io's "channels" for multiple make chat teams to exist
1. Using Socket.io's "rooms" to make themed chat rooms inside a channel, like "general" and "random"
1. Dropping the client into an Electron app to make a desktop app.
1. Adding a Team and a User resource and persisting teams and authenticating users.
1. Great article on [Socket.io's strengths and weaknesses](https://dzone.com/articles/socketio-the-good-the-bad-and-the-ugly)

You have all the tools at your disposal to add these enhancements, we believe in you!
