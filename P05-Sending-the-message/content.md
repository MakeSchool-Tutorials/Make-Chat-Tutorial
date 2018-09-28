---
title: Start chatting
slug: Sending-the-message
---

## Make it look like Slack

Now that our server and client are ready for new users, lets update the look of our website.

I'll give you the new handlebars and CSS.

```html
<!--index.handlebars-->
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

    <div class="mainContainer">
      <div class="channelsAndUsersOnlineContainer">
        <h1 class="brand">Make Chat</h1>
        <div class="channels">
            <h2 class="channelsLabel">Channels</h2>
            <div class="newChannelForm"><input id="newChannelInput" placeholder="New Channel" /><button id="newChannelBtn">Create</button></div>
            <div class="channel-current">General</div>
        </div>
        <h2>Online Users</h2>
        <div class="usersOnline"></div>
      </div>
      <div class="chatContainer">
        <div class="chatContainerFluid">
          <div class="messageContainer">
              <h2>Messages</h2>
          </div>
          <div class="textChatDivide"></div>
          <div class="chatBox"><textarea id="chatInput" placeholder="Type a message"></textarea><button class="btn" id="sendChatBtn">Send</button></div>
        </div>
      </div>
    </div>

  </body>
</html>
```

```css
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

.channels {
  display: flex;
  flex : 6;
  flex-direction: column;
  margin-right: 15px;
  overflow-y: auto;
  width: 100%;
}

.newChannelForm{
  display: flex;
  margin-top: -10px;
  margin-bottom: 10px;
  margin-left: 8px;
  margin-right: 8px;
}

#newChannelInput{
  font-size: 14px;
  width : 70%;
}

#newChannelBtn{
  font-size: 14px;
  width: 30%;
}

.channel{
  font-size: 18px;
  padding : 10px 0px;
  margin : 2px 0px;
  cursor : pointer;
}

.channel:hover{
  color : grey;
}

.channel-current{
  font-weight: bold;
  font-size: 18px;
  padding : 10px 0px;
  margin : 2px 0px;
  background-color: #4f9689;
}

.brand{
  margin-left: 8px;
}

.channelsLabel{
  margin-left: 8px;
}


.chatContainer {
  width : 80%;
  display: flex;
  flex-direction: column;
  background-color: white;
}

.chatContainerFluid{
  margin : 8px;
  display: flex;
  flex-direction: column;
}

.channelsAndUsersOnlineContainer{
  width : 20%;
  margin-left : -8px;
  margin-top: -8px;
  margin-right: 8px;
  display: flex;
  flex-direction: column;
  height : 750px;
  background-color: #4d394b;
  color : white;
}

.usersOnline{
  margin-left: 8px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: flex-start;
  overflow-y: auto;
  flex : 4;
}

.userOnline{
  margin-bottom: 8px;
  font-size: 18px;
}

.textChatDivide{
  border : 0.5px solid green;
  margin-bottom: 10px;
}

.chatBox {
  display: flex;
  justify-content: center;
  width : 100%;
}

label {
  display: block;
}

#chatInput {
  width : 100%;
  display: inline;
}

#sendChatBtn{
  width : 10%;
  display: inline;
}

.messageContainer{
  display: flex;
  margin-bottom: 10px;
  padding-bottom: 10px;
  flex-direction: column;
}

.message{
  margin-bottom: 5px;
}

.messageUser{
  display: inline;
  color : red;
  font-weight: bold;
}
.messageText{
  display: inline;
}

.mainContainer{
  display: none;
  flex-direction: row;
  width : 100%;
}
```

Also update your `index.js` accordingly.

```javascript
//index.js
$(document).ready(()=>{
  const socket = io.connect();

  $('#createUserBtn').click((e)=>{
    e.preventDefault();
    if($('#usernameInput').val().length > 0){
      socket.emit('new user', $('#usernameInput').val());
      $('.usernameForm').remove();
      // Have the main page visible
      $('.mainContainer').css('display', 'flex');
    }
  });

  //socket listeners
  socket.on('new user', (username) => {
    console.log(`${username} has joined the chat`);
    // Add the new user to the online users div
    $('.usersOnline').append(`<div class="userOnline">${username}</div>`);
  })

})
```

Now if you enter a username, you will see the main content, as well as the online users.

# Let's send some messages

Look at the nice text area we have for sending messages. If only it worked...

```javascript
//index.js
$(document).ready(()=>{
  const socket = io.connect();

  //Keep track of the current user
  let currentUser;

  $('#createUserBtn').click((e)=>{
    e.preventDefault();
    if($('#usernameInput').val().length > 0){
      socket.emit('new user', $('#usernameInput').val());
      // Save the current user when created
      currentUser = $('#usernameInput').val();
      $('.usernameForm').remove();
      $('.mainContainer').css('display', 'flex');
    }
  });

  $('#sendChatBtn').click((e) => {
    e.preventDefault();
    // Get the message text value
    let message = $('#chatInput').val();
    // Make sure it's not empty
    if(message.length > 0){
      // Emit the message with the current user to the server
      socket.emit('new message', {
        sender : currentUser,
        message : message,
      });
      $('#chatInput').val("");
    }
  });

  //socket listeners
  socket.on('new user', (username) => {
    console.log(`${username} has joined the chat`);
    $('.usersOnline').append(`<div class="userOnline">${username}</div>`);
  })

})
```

A few things were added here. We are now saving the client's username to the `currentUser` variable.

We then make sure our send chat button **emits** `"new message"` to our server. As shown, you can emit multiple pieces of data.

In this case we are emitting `sender` and `message`.

Let's now create the `"new message"` listener on our server.

```javascript
//chat.js
module.exports = (io, socket) => {

  socket.on('new user', (username) => {
    console.log(`✋ ${username} has joined the chat! ✋`);
    io.emit("new user", username);
  })

  //Listen for new messages
  socket.on('new message', (data) => {
    // Send that data back to ALL clients
    console.log(`🎤 ${data.sender}: ${data.message} 🎤`)
    io.emit('new message', data);
  })

}
```

Simple enough. Notice how we can access the `data` sent like an object.

We should update the client to receive and show new messages from the server.

```javascript
//index.js

//socket listeners
socket.on('new user', (username) => {
  console.log(`${username} has joined the chat`);
  $('.usersOnline').append(`<div class="userOnline">${username}</div>`);
})

//Output the new message
socket.on('new message', (data) => {
  $('.messageContainer').append(`
    <div class="message">
      <p class="messageUser">${data.sender}: </p>
      <p class="messageText">${data.message}</p>
    </div>
  `);
})
```

Wow, what a lot of work. See if the page updates for each message.

# Great, we have users and chat! We're finished right?

Not so fast, Sonic.

If you were to reload or create a new instance in another tab, you will see that the online users and messages don't save.

This is because our application is currently only showing new data coming in. There is no system in place to show old data.

Also we don't even have different channels. How boring is that?

Let's move on!
