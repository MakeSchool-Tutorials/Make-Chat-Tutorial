---
title: "I'm Ready to Get Started"
slug: get-it-together
---

You know the drill by now. But in case you forgotten the Node essentials...

```bash
$ mkdir your-project-name
$ cd your-project-name
$ npm init
```

When your `package.json` arrives, do yourself a favor and make sure it has these module dependencies.

Also, create an `app.js` while you're at it.

```bash
$ npm install express mongoose express-handlebars --s
$ touch app.js
```

Here's what your `app.js` should look like.

```javascript
//App.js
const express = require('express');
const app = express();
//Socket.io has to use the http server
const server = require('http').Server(app);

//Express View Engine for Handlebars
const exphbs  = require('express-handlebars');
app.engine('handlebars', exphbs());
app.set('view engine', 'handlebars');

app.get('/', (req, res) => {
  res.render('index.handlebars');
})

server.listen('3000', () => {
  console.log('Server listening on Port 3000');
})
```

# Add in the views

If you tried running the server, you'll notice you haven't yet created index.handlebars.
Let's do that.

```bash
$ mkdir views
$ cd views
$ touch index.handlebars
$ cd ..
```

```html
<!--index.handlebars-->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title></title>
  </head>
  <body>
    <h1>Socket.io</h1>
  </body>
</html>
```

# Looking Pretty Good!

Time to put on your big kid gear, because you're now about to make your first websocket connection!
