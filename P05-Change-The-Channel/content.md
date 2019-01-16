---
title: "Saving Data and Channels"
slug: change-the-channel
---

## Be a Hero and Save These Users!

Don't worry we're not building a database for this application.

We'll save everything locally, meaning that all of our data will be cleared on a server restart.

Let's start with updating our app.js

```javascript
//app.js
const io = require('socket.io')(server);
//We'll store our online users here
let onlineUsers = {};
io.on("connection", (socket) => {
  // Make sure to send the users to our chat file
  require('./sockets/chat.js')(io, socket, onlineUsers);
})
```

*Why are we using an object instead of an array to save our users?*

Great question! This object will actually act as a dictionary to access each user's ID by their username.

*ID? What ID are you talking about?*

A socket id of course! This is going be helpful for when we implement private messages. In the meantime, go to your chat.js

```javascript
//chat.js
module.exports = (io, socket, onlineUsers) => {

  socket.on('new user', (username) => {
    //Save the username as key to access the user's socket id
    onlineUsers[username] = socket.id;
    //Save the username to socket as well. This is important for later.
    socket["username"] = username;
    console.log(`✋ ${username} has joined the chat! ✋`);
    io.emit("new user", username);
  })

  socket.on('new message', (data) => {
    console.log(`🎤 ${data.sender}: ${data.message} 🎤`)
    io.emit('new message', data);
  })

}
```

Back to your client, let's ask to see all the online users when the page loads.
```javascript
//index.js
const socket = io.connect();
let currentUser;
// Get the online users from the server
socket.emit('get online users');
```

Now update chat.js to send the online users.
```javascript
//chat.js
socket.on('get online users', () => {
  //Send over the onlineUsers
  socket.emit('get online users', onlineUsers);
})
```

Finally go back to your client, to show the users on the page.
```javascript
//index.js

//socket listeners
socket.on('get online users', (onlineUsers) => {
  //You may have not have seen this for loop before. It's syntax is for(key in obj)
  //Our usernames are keys in the object of onlineUsers.
  for(username in onlineUsers){
    $('.usersOnline').append(`<p class="userOnline">${username}</p>`);
  }
})
```

Go ahead and test this out in multiple tabs and see that all users save, regardless of reload.

The only problem is that users are not leaving. This is an easy fix.

# Be a villain and destroy these users!

When a user closes out of the application, we want to get rid of them.

Go to your chat.js

```javascript
//chat.js

//This fires when a user closes out of the application
socket.on('disconnect', () => {
  //This deletes the user by using the username we saved to the socket
  delete onlineUsers[socket.username]
  io.emit('user has left', onlineUsers);
});
```

**socket.on("disconnect")** is a special listener that fires when a user exits out of the application.

Now update the client to be **on** when a *"user has left"*.

```javascript
//index.js

//Refresh the online user list
socket.on('user has left', (onlineUsers) => {
  $('.usersOnline').empty();
  for(username in onlineUsers){
    $('.usersOnline').append(`<p>${username}</p>`);
  }
});
```

# Channel Your energy

Wow you made it this far. I hope you've been taking breaks because this next part is a bit much.

We're now going to create different channels for users to create or join. Let's add some functionality to the channel creator on the client.

```javascript
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

## Go to your room!

Socket.io has this fancy thing called **rooms** in which different sockets(clients) can **join**.

In the code we just put in, we did **socket.join(newChannel)** which is telling the socket to join the new channel room.

Rooms are great, because they allow for unique **emits**. You will see what I mean very soon.

Let's update the client.

```javascript
//index.js

//Add the new channel to the channels list (Fires for all clients)
socket.on('new channel', (newChannel) => {
  $('.channels').append(`
    <div class="channel">${newChannel}</div>
  `);
});

//Make the channel joined the current channel. Then load the messages.
//This only fires for the client who made the channel.
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

We should now update the *"new message"* listener on the server.

```javascript
//chat.js
socket.on('new message', (data) => {
  //Save the new message to the channel.
  channels[data.channel].push({sender : data.sender, message : data.message});
  //Emit only to sockets that are in that channel room.
  io.to(data.channel).emit('new message', data);
});
```

And update the *"new message"* listener on the client
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

Let's add a listener for *"user changed channel"*

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

Extra Credit is implementing a private message system. You have all the tools you need to do this, I believe in you!
