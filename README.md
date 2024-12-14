# Talkomatic Bot Guide

This guide will teach you how to create your own little bot in Talkomatic Classic and Talkomatic Modern!

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

How about let's make the bot join a room? Replace `000000` with your testing room ID.
```js
socket.on("connect", async () => {
  console.log("Socket connected!");
  
  socket.emit("join lobby", {
    username: "TutorialBot",
    location: "Somewhere"
  });
  
  socket.emit("join room", {
    roomId: "000000"
  });
});
```

Recap! This might look a bit daunting, but don't worry. First, the bot listens for when it connects to the socket, joins the lobby with its user info, then joins a room at a set room ID.

Great! Now let's check when the bot joins, and log it. Append this **after** your `connect` event:
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

### Listening to Users

Talkomatic Classic has a unique way of storing user messages. Talkomatic Classic likes to use a "diff" method, in which it tells the client what has been replaced, deleted, added, etc.

At the top of your program under the `const socket` line, initialize a `users` object where we will store user data and typing data:

```js
var users = {};
```

We should listen for when users leave, join, and when our bot initializes. Replace the `room joined` listener with:
```js
socket.on("room joined", async (data) => {
  sendMessage("Hello, world!");

  users = Object.fromEntries(data.users.map(user => {
    if (!user.username) user.username = "Anonymous";
    if (!user.location) user.location = "Unknown";

    user.typing = data.currentMessages[user.id];
    return [user.id, user];
  }));
});
```
This takes the initialization data the server sends us and appends user data to our `users` object for each user.

**Under** this event, we can listen to user joins and leaves as well:
```js
socket.on("user joined", async (user) => {
  delete user.roomName;
  delete user.roomType;
  user.typing = "";
  users[user.id] = user;
});
socket.on("user left", async (user_id) => {
  const user = users[user_id];
  if (!user) return;
  
  delete users[user.id];
});
```
The `user joined` listener removes some unnecessary data then also initializes the typing and appends it to the existing `users` object. On the other hand, the `user left` listener checks if the user is still in the room, and if so, removes the user.

The users have just been set up, but we still need to see the messages users are sending. This task isn't easy and requires you to know how to the diff method actually works. Insert this huge chunk after the `user left` listener.
```js
socket.on("chat update", async (data) => {
  const author = users[data.userId];
  if (!author) return;

  const prev_content = author.typing;

  if (data.diff) {
    if (data.diff.type == "full-replace") {
      author.typing = data.diff.text;
    } else {
      const cur_text = author.typing;
      var new_text;

      switch (data.diff.type) {
        case "add":
          new_text = cur_text.slice(0, data.diff.index) + data.diff.text + cur_text.slice(data.diff.index);
          break;
        case "delete":
          new_text = cur_text.slice(0, data.diff.index) + cur_text.slice(data.diff.index + data.diff.count);
          break;
        case "replace":
          new_text = cur_text.slice(0, data.diff.index) + data.diff.text + cur_text.slice(data.diff.index + data.diff.text.length + 1);
          break;
        default:
          new_text = cur_text;
          break;
      }

      author.typing = new_text;
    }
  } else {
    author.typing = data.message;
  }
});
```
Whoa! What does this all do, and why so long? First, the bot gets the user from the `users` object, and stops if it isn't in the room. Then it checks the diff method, the easist being `full-replace`, where all you need to do is get the text and you're done. However, diff types like `add`, `delete`, and `replace` all require the knowledge of the previous message. `add` will add text at an index, same with `delete`, and similar to `replace`. If there is an issue it will default to either the current text or the message currently being analyzed.

### Creating Commands

Coming soon!

#### Bot guide created with :heart: by @xnor / ZackiBoiz
