---
title: "Saving and Destroying Users"
slug: saving-and-destroying-users
---

1. ~~Build out a basic view~~
1. ~~Integrate sockets~~
1. ~~Implement user form~~
1. ~~Style and send messages~~
1. **Connect/disconnect users**
    1. **Save the `onlineUsers` object**
    1. **Display the online users**
    1. **Remove users when they leave the app**
1. Create/persist/join channels

Don't worry we're not building a database for this application.

We'll save everything locally, meaning that all of our data will be cleared on a server restart.

# Be a Hero and Save These Users!

Let's start with updating our `app.js` to save the object `onlineUsers`.

>[action]
> Update the `socket.io` code in `app.js` to the following:
>
```javascript
const io = require('socket.io')(server);
//We'll store our online users here
let onlineUsers = {};
io.on("connection", (socket) => {
  // Make sure to send the users to our chat file
  require('./sockets/chat.js')(io, socket, onlineUsers);
})
```

*Why are we using an object instead of an array to save our users?*

Great question! This object will actually act as a **dictionary** to access each user's ID by their username.

*ID? What ID are you talking about?*

A socket ID of course! Each socket has a unique ID that identifies it as a unique connected user. So we can make this ID do double duty to identify our users. This is going be helpful for when we implement private messages.

In the meantime, let's update `chat.js` to save those `onlineUsers`.

> [action]
> Update `/sockets/chat.js` to the following code to save the `onlineUsers`:
>
```javascript
//chat.js
module.exports = (io, socket, onlineUsers) => {
>
  socket.on('new user', (username) => {
    //Save the username as key to access the user's socket id
    onlineUsers[username] = socket.id;
    //Save the username to socket as well. This is important for later.
    socket["username"] = username;
    console.log(`✋ ${username} has joined the chat! ✋`);
    io.emit("new user", username);
  })
>
  socket.on('new message', (data) => {
    console.log(`🎤 ${data.sender}: ${data.message} 🎤`)
    io.emit('new message', data);
  })
>
}
```

Back to your client, let's ask to see all the online users when the page loads.

>[action]
> Add the `socket.emit` line right below `let currentUser` in `/public/index.js`:
>
```javascript
//index.js
const socket = io.connect();
let currentUser;
// Get the online users from the server
socket.emit('get online users');
```
>
> Now update `/sockets/chat.js` to include code to send the online users when someone connects.
>
```javascript
...
>
socket.on('get online users', () => {
  //Send over the onlineUsers
  socket.emit('get online users', onlineUsers);
})
```

Finally go back to your client, to show the users on the page.

>[action]
> Update `/public/index.js` to include a `get online users` socket listener:
>
```javascript
...
>
socket.on('get online users', (onlineUsers) => {
  //You may have not have seen this for loop before. It's syntax is for(key in obj)
  //Our usernames are keys in the object of onlineUsers.
  for(username in onlineUsers){
    $('.users-online').append(`<div class="user-online">${username}</div>`);
  }
})
```

Go ahead and test this out in multiple tabs and see that all users save, regardless of reload.

The only problem is that users are not leaving when you close a window. The fix is pretty straightforward, we'll use the built in `socket.on('disconnect')` event to emit that someone left.

# Be a Villain and Destroy These Users!

When a user closes out of the browser window, we want to get rid of them from our `onlineUsers`.

**Update `/sockets/chat.js` to include a `disconnect` listener:**

**Hints:**

- there is a [delete](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/delete) function...
- There's also a [disconnect](https://socket.io/docs/client-api/#socket-disconnect) listener...

>[solution]
>
```javascript
...
>
// This fires when a user closes out of the application
// socket.on("disconnect") is a special listener that fires when a user exits out of the application.
socket.on('disconnect', () => {
  //This deletes the user by using the username we saved to the socket
  delete onlineUsers[socket.username]
  io.emit('user has left', onlineUsers);
});
```

Now update the client to **refresh its online users** when a *"user has left"*.

**Update `/public/index.js` to include a `user has left` listener:**

>[solution]
>
```javascript
...
>
//Refresh the online user list
socket.on('user has left', (onlineUsers) => {
  $('.users-online').empty();
  for(username in onlineUsers){
    $('.users-online').append(`<p>${username}</p>`);
  }
});
```

# Test, Test, Test

Open up multiple browser windows and test joining and leaving the chat.

# Now Commit

```bash
$ git add .
$ git commit -m 'Users can join and leave chats'
$ git push
```

# Feedback and Review - 2 minutes

**We promise this won't take longer than 2 minutes!**

Please take a moment to rate your understanding of learning outcomes from this tutorial, and how we can improve it via our [tutorial feedback form](https://goo.gl/forms/L3i5ZhY58AOGtyqI3)

# Stretch Challenge: Logout

>[challenge]
> Can you create a "log out" button, that breaks your connection, removes you from `onlineUsers`, and redirects you back to the seeing the login form?
