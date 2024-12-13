# Talkomatic-Bot-Guide

This guide will teach you how to create your own little bot in bot Talkomatic Classic and Talkomatic Modern!

## Talkomatic Classic Guide

### Getting Started

This tutorial teaches you how to create a bot using `Node.js`, but you can use any language that supports the use of `WebSockets`!

Create your project and install `Socket.IO` so we can interact with `WebSockets`:
```sh
npm i socket.io-client
```
Make a new file that we will call `index.js`. This is where our bot's code will be.

We can start with requiring the module, and then connecting to Talkomatic Classic's socket.

```js
const io = require("socket.io-client");

const socket = io("https://open.talkomatic.co");
socket.on("connect", async () => {
  console.log("Socket connected!");
});
```

Test this by running:
```sh
node index.js
```
You should see "Socket connected!" in the console! We have just connected to Talkomatic's servers.

### Creating a Bot

Now, we need to have the bot setup its profile. (Username, location, etc.) Let's do that with:
```js
socket.on("connect", async () => {
  console.log("Socket connected!");
  
  socket.emit("join lobby", {
    username: "TutorialBot",
    location: "Somewhere"
  });
});
```
You won't see your bot anywhere, but the bot has just connected to the lobby. When the bot connects to the socket, it will ask the server to join the lobby by setting its username and location. You can customize this!

How about let's make the bot create a room and join it after it joins the lobby?
```js
socket.on("connect", async () => {
  console.log("Socket connected!");
  
  socket.emit("join lobby", {
    username: "TutorialBot",
    location: "Somewhere"
  });
  
  socket.once("room created", (room_id) => {
    socket.emit("join room", {
      roomId: room_id
    });
  });
  socket.emit("create room", {
    name: "Tutorial Room",
    type: "public",
    layout: "vertical"
  });
});
```

This might look a bit daunting, but don't worry. First, the bot listens for when the room is created, then joins the room with the room ID sent back by the server. Then it actually asks the server to create the room with the name, type, and layout. You may also add an optional `accessCode` parameter if your type is `semi-private`, but will have to have that same `accessCode` parameter when you join the room.

All types are `public`, `semi-private`, and `private`. All layouts are `vertical` and `horizontal`.

Great! Now let's check when the bot joins, and log it. Append this after your `connect` event:
```js
socket.on("room joined", (data) => {
  console.log("Joined room!");
  console.log(data);
});
```
The server will return some data along when you join the room. We can use this to save user messages, votes, etc.

### Sending Messages

Awesome sauce! Now we have our bot in a room, but it isn't doing anything right now. It's just sitting there. After our bot joins the room, let's have it say the iconic phrase, "Hello, world!"

There are four main message types: `full-replace`, `replace`, `delete`, and `add`. For bots it is easiest to just use `full-replace` as we don't need to calculate anything extra. Let's do that:

```js
socket.on("room joined", (data) => {
  console.log("Joined room!");
  console.log(data);
  
  socket.emit("chat update", {
    diff: {
      type: "full-replace",
      text: "Hello, world!"
    }
  });
});
```
This event sends a `chat update` request that basically tells the server to replace our entire chat box with "Hello, world!". Try it!

Now that we know how to send messages, let's make a simple function that does that so we can reuse it easily. Define this at the bottom of your program:

```js
function sendMessage(text) {
  socket.emit("chat update", {
    diff: {
      type: "full-replace",
      text: text
    }
  });
}
```
That also means we can replace the join message with:

```js
sendMessage("Hello, world!");
```

### Will continue the guide later!

#### Bot guide created with :heart: by @xnor / ZackiBoiz
