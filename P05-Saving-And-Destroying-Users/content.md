---
title: "Saving and Destroying Users"
slug: saving-and-destroying-users
---

Don't worry we're not building a database for this application.

We'll save everything locally, meaning that all of our data will be cleared on a server restart.

# Be a Hero and Save These Users!

Let's start with updating our `app.js` to save the object `onlineUsers`.

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

A socket id of course! Each socket has a unique id that identifies it as a unique connected user. So we can make this id do double duty to identify our users. This is going be helpful for when we implement private messages.

In the meantime, let's update your `chat.js` to save those `onlineUsers`.

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

Now update `chat.js` to send the online users when someone connects.

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

The only problem is that users are not leaving when you close a window. The fix is pretty straightforward, we'll use the built in `socket.on('disconnect')` event to emit that someone left.

# Be a Villain and Destroy These Users!

When a user closes out of the browser window, we want to get rid of them from our `onlineUsers`.

Go to your `chat.js`

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

# Test, Test, Test

Open up multiple browser windows and test joining and leaving the chat.

# Stretch Challenge: Logout

Can you create a "log out" button, that breaks your connection, removes you from `onlineUsers`, and redirects you back to the seeing the login form?
