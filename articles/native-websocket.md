# Using the browsers native WebSocket API

Web Sockets have existed in modern browsers since 2012. They allow communication from the browser to the server and also the other way around! Pretty cool right?
This makes updating realtime data in the browser easier as you don’t have to fire a new fetch request every so many seconds or minutes.

This article isn’t going to use socket.io as I didn’t feel the need to use a library for this since the browser has it’s own webSocket API which works quite well.

## Server code
Let’s look at the server code required for this. We are essentially setting up a second server which is the web socket server.
First we need to install the `ws` package from npm.
```sh
npm install ws
```

Since it’s a second server it’s a good idea to build it in a separate file, called `websocket.js`.

Let’s create the server.
```js
const WebSocketServer = require('ws').Server
const wss = new WebSocketServer({ port: 40510 })
```
This creates a web socket server on port 40510.

Now we need to add some code to do something when a client connects.
```js
const WebSocketServer = require('ws').Server
const wss = new WebSocketServer({ port: 40510 })

wss.on('connection', ws => {
  ws.on('message', message => {
    console.log('received:', message)
  })

  ws.send('Hello world!')
})
```
This listens for a connection. If a client connects and sends a message, it gets logged to the console and a message saying `Hello world!` gets sent to the client.

If you also want to do something when a connection closes you can add the following bit of code.
```js
wss.on('close', ws => {
  console.log('Websocket disconnected')
})
```
However, this code only interacts with connected clients individually. 

If you want to broadcast something to all connected clients you can add the following bit.
```js
wss.broadcast = msg => {
  wss.clients.forEach(client => {
    client.send(msg)
  })
}
```

### Sending from express
With all the code above, everything happens within the `web socket.js` file. If you want to interact with all connected clients from within express you can do the following.

```js
// websocket.js
module.exports = wss

// app.js
const wss = require('path/to/websocket.js')

wss.broadcast('This is a broadcasted message')
```

## Client code
For the client code we aren’t going to use any libraries. Just the native WebSocket API.

```js
if ('WebSocket' in window) {
	const socket = new WebSocket('ws://localhost:40510')

	socket.addEventListener('message', event => {
		const data = event.data
		
		// Do something with the event and/or the received data.
	}
}
```
This code does the following:
1. Check if the WebSocket API is available in the browser
2. Create a new WebSocket
3. Listen for a message from the server

And that’s basically it! These are the basics of using the WebSocket API.