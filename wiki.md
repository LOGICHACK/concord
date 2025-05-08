Coding partner

Show thinking
Okay, creating documentation is a crucial step for making your extension system usable! Here is a Wiki page draft in Markdown format based on the botAPI we defined in your server.js. You can copy and paste this into a file (e.g., API_WIKI.md) or directly into a GitHub Wiki page.

Markdown

# Haven Chat Extension API Documentation

Welcome to the documentation for the Haven Chat Extension API! This guide will help you create extensions (bots) that can interact with your Haven Chat server.

## Getting Started

### Extension Structure

Extensions are JavaScript modules loaded by the main server from the `/extensions` directory at startup. Each extension should be either:

1.  A single `.js` file (e.g., `myBot.js`).
2.  A directory containing an `index.js` or `main.js` file (e.g., `/myComplexBot/index.js`).

An extension module **must** export an object with an `initialize` function. It can optionally export a `name`.

**Example Extension File (`/extensions/helloBot.js`):**

```javascript
// extensions/helloBot.js
module.exports = {
  // Optional: If not provided, the filename (without .js) is used.
  name: "HelloBot",

  // Required: Called by the server on load. Receives the botAPI object.
  initialize: (botAPI) => {
    botAPI.log("Initialization complete!");

    // Register commands or set up listeners here
    botAPI.registerCommand("greet", (args, context) => {
        botAPI.sendMessage(context.channel, `Hi ${context.sender}!`);
    }, "Sends a simple greeting.");
  },

  // Optional: Called for every non-command message AFTER it's saved
  onMessage: (message, botAPI) => {
    // botAPI.log(`Received message in #${message.channel}: ${message.text}`);
  },

  // Optional: Called BEFORE a message is processed for commands or saved
  beforeMessage: async (messageContext, botAPI) => {
    // botAPI.log(`Processing message from ${messageContext.sender}: ${messageContext.text}`);
    // Return false to block, string to modify, true/undefined to allow.
    return true;
  },

  // Optional: Called when a user disconnects or leaves a channel they were in
  onUserLeave: (leaveContext, botAPI) => {
     // botAPI.log(`User left: ${leaveContext.username}`);
  }
}; ```


Loading
The server automatically loads all valid extensions from the /extensions directory when it starts. Check the server console logs for loading status and potential errors.

The botAPI Object
When your extension's initialize function is called, it receives a botAPI object. This object is your interface to the chat server.

API Reference
Here are the properties and methods available on the botAPI object:

General
extensionName (String)
Read-only property containing the name of your extension (either from module.exports.name or the filename).
log(message)
Logs a message to the main server console, prefixed with your extension's name. Useful for debugging.
Parameters:
message (Any): The message or object to log.
Message Sending
sendMessage(channelName, text, options)
Sends a message to the specified channel. Currently only supports channels of type chat or announcements.
Parameters:
channelName (String): The name of the channel to send to (e.g., "general").
text (String): The plain text content of the message.
options (Object, Optional): An object containing message options:
senderName (String, Optional): Override the bot's sender name (defaults to extensionName).
avatarUrl (String, Optional): URL for the bot's avatar for this message.
avatarText (String, Optional): Fallback text for the avatar (defaults to initials of senderName).
avatarBg (String, Optional): Tailwind background class for the avatar (defaults to bg-teal-500).
embed (Object, Optional): An embed object to include with the message (see Embed Object Structure below).
Embed Object Structure
If you provide an embed object in the sendMessage options, the client (if updated) can render a rich embed. The structure is:

JavaScript

{
  title: "String (Max 256 chars recommended)",
  description: "String (Markdown supported client-side, max 2048 chars recommended)",
  color: "#HexColorString (e.g., '#FF00AA')", // Optional
  fields: [ // Optional array of field objects
    {
      name: "String (Field Title, max 256 chars)",
      value: "String (Field Value, Markdown supported client-side, max 1024 chars)",
      inline: Boolean // Optional: Whether the field should display inline with others (defaults to false)
    }
    // ... more fields (max ~25 fields recommended)
  ],
  thumbnailUrl: "String (URL of a small image appearing near the title)", // Optional
  imageUrl: "String (URL of a larger image appearing below the embed content)", // Optional
  footer: { // Optional footer object
    text: "String (Footer text, max 2048 chars)",
    iconUrl: "String (URL of a small icon next to footer text)" // Optional
  }
  // timestamp: "ISO 8601 String" // Optional: Timestamp shown near footer (server might add automatically)
}
Note: The client (havenchat.html) needs specific code added to parse and render these embed objects.

Command Handling
registerCommand(command, handler, description)
Registers a chat command (prefixed with !) that your extension will handle.
Parameters:
command (String): The name of the command without the prefix (e.g., "hello", "play-card").
handler (Function): The function to execute when the command is invoked. It receives (args, messageContext, botAPI) as arguments.
description (String, Optional): A brief description of the command (for future help commands).
Handler Function Parameters:
args (Array&lt;String>): An array of arguments passed after the command (split by spaces).
messageContext (Object): Information about the message that triggered the command:
sender (String): Username of the user who sent the command.
senderAvatarUrl (String): Avatar URL of the sender.
senderRoles (Array&lt;String>): Roles of the sender (e.g., ['user'], ['admin', 'user']).
text (String): The full, original message text.
channel (String): The name of the channel where the command was used.
messageId (String): The unique ID of the command message.
timestamp (String): ISO 8601 timestamp of the message.
socketId (String): The internal socket ID of the sender's connection (for advanced use).
botAPI (Object): The same bot API instance, passed for convenience within the handler.
Data Storage
Extensions can store and retrieve simple data associated with themselves using keys. Data is stored as JSON files in chat_data/extension_data/<YourExtensionName>/.

storeData(key, value) (Async)

Saves value associated with the given key. The value will be JSON stringified.
Parameters:
key (String): The identifier for the data (e.g., "userCollections", "mtgGameState_123"). Used as the filename (key.json).
value (Any Serializable): The data to store.
Returns: (Promise&lt;Boolean>): Resolves true on success, false on failure.
retrieveData(key) (Async)

Retrieves data previously stored with storeData.
Parameters:
key (String): The identifier for the data.
Returns: (Promise&lt;Any | null>): Resolves with the parsed data, or null if not found or an error occurred.
Information Access
getChannelData(channelName)
Gets basic information about a specific channel.
Parameters:
channelName (String): The name of the channel.
Returns: (Object | null): An object { name, type, isPubliclyWritable, messageCharLimit } or null if not found.
getUsersInChannel(channelName)
Gets a list of usernames currently connected to a specific channel room. (Note: Presence depends on users having joined the channel via joinChannel).
Parameters:
channelName (String): The name of the channel.
Returns: (Array&lt;String>): An array of usernames.
getUserData(username)
Gets basic information about a specific user.
Parameters:
username (String): The username to look up.
Returns: (Object | null): An object { username, roles, avatarUrl } or null if not found.
Moderation
deleteMessage(channelName, messageId)
Attempts to delete a message from a channel (chat messages or thread replies). Note: Deleting thread OP messages is not supported via this simple call. Permissions are currently not checked by this API call itself; any extension can call it. Implement permission checks within your extension logic using messageContext.senderRoles if needed.
Parameters:
channelName (String): The channel where the message exists.
messageId (String): The ID of the message/reply to delete.
Returns: (Boolean): true if a message was found and deleted, false otherwise.
Advanced
getIO()
Returns the raw Socket.IO server instance (io).
Warning: This provides unrestricted access to the underlying Socket.IO functionality. Use with extreme caution, as misuse could easily break the chat server or bypass intended API limitations. Prefer using the specific botAPI methods whenever possible.
Extension Event Hooks
Your extension module can optionally export functions to hook into server events:

initialize(botAPI) (Required)
Called once when the extension is loaded. Use this to set up listeners, register commands, etc.
onMessage(message, botAPI) (Optional)
Called for every message (user messages, bot messages, thread OP messages, thread replies) after it has been processed by beforeMessage, checked for commands (if applicable), saved, and emitted to clients.
Parameters:
message (Object): The final message object. Includes sender, text, channel, messageId, timestamp, avatarUrl, isBotMessage, userRoles, and potentially threadId, isOpMessage.
botAPI (Object): Your API instance.
beforeMessage(messageContext, botAPI) (Optional, Async supported)
Called before a user message is processed for commands or saved. Ideal for filtering, modifying content, or potentially handling interactions before the standard flow.
Parameters:
messageContext (Object): Similar to command context - { sender, senderAvatarUrl, senderRoles, text, channel, messageId, timestamp, socketId }. Note messageId and timestamp are pre-generated here.
botAPI (Object): Your API instance.
Return Value:
false: Blocks the message entirely. messageError is sent to the originating user.
String: Replaces the original message text with the returned string. Processing continues with the modified text.
true / undefined / (Any other value): Allows the message to proceed unmodified.
onUserLeave(leaveContext, botAPI) (Optional)
Called when a user disconnects from the server while logged in.
Parameters:
leaveContext (Object): Information about the user leaving: { username, channel } (channel is the last channel they were known to be in).
botAPI (Object): Your API instance.
