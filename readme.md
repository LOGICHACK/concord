# **Concord Server API Documentation**

This document outlines the APIs available for developing Vines and Grape Modules for the Concord chat server.


## **VineAPI**

The VineAPI provides methods for Vines to interact with the Concord server, such as sending messages, registering commands, and managing data. A Vine gets its own API instance when its initialize function is called.


### **API Methods**

- **vineName**:

* Description: The name of the Vine this API instance belongs to.

* Returns: string

- **log(message)**:

* Description: Logs a message to the server console, prefixed with the Vine's name.

* Parameters:

- message (string): The message to log.

* **sendMessage(channelName, text, options = {})**:

- Description: Sends a message to a specified chat channel. **Important:** This method currently relies on the DefaultChatGrape module being loaded and the target channel being managed by it. It sends a message formatted as a bot message.

- Parameters:

* channelName (string): The name of the target channel (must be a DefaultChatGrape channel).

* text (string): The message content.

* options (object, optional): Additional message options.

- senderName (string): Custom sender name for the bot message (defaults to the vine name).

- avatarUrl (string): Custom avatar URL for the bot message.

- avatarText (string): Custom avatar text (defaults to first 2 chars of sender name).

- avatarBg (string): Custom avatar background CSS class (defaults to bg-purple-600).

- embed (object): Embed object to include with the message (structure depends on client implementation).

* **registerCommand(command, handler, description = "No description")**:

- Description: Registers a slash command (/command) that users can invoke in chat. If a command is already registered, it will be overwritten. The handler function is executed when the command is used.

- Parameters:

* command (string): The command name (without the leading /).

* handler (function): The asynchronous function to execute. Receives (socket, args, channelName, vineAPI) as arguments.

- socket: The user's socket object.

- args (Array): An array of arguments following the command.

- channelName (string): The channel where the command was invoked.

- vineAPI: The API instance for the Vine that registered the command.

* description (string, optional): A brief description of the command (currently not displayed to users but good practice).

- **storeData(key, value)**:

* Description: Asynchronously stores JSON-serializable data specific to this Vine. Data is saved in the chat\_data/vine\_data/\<vineName>/ directory.

* Parameters:

- key (string): The key (filename without extension) to store the data under.

- value (any): The JSON-serializable data to store.

* Returns: Promise\<boolean> (true on success, false on error).

- **retrieveData(key)**:

* Description: Asynchronously retrieves previously stored data for this Vine.

* Parameters:

- key (string): The key of the data to retrieve.

* Returns: Promise\<any | null> (the retrieved data, or null if not found or on error).

- **getChannelData(channelName)**:

* Description: Gets a deep copy of the data for a specific channel from the server's main channel store.

* Parameters:

- channelName (string): The name of the channel.

* Returns: object | null (a copy of the channel data, or null if the channel doesn't exist).

- **getUsersInChannel(channelName)**:

* Description: Gets a list of usernames currently connected to a specific channel's socket.io room.

* Parameters:

- channelName (string): The name of the channel.

* Returns: Array\<string> (list of usernames currently in the channel's room).

- **getUserData(username)**:

* Description: Gets a deep copy of the data for a specific user from the server's main user store.

* Parameters:

- username (string): The username.

* Returns: object | null (a copy of the user data, or null if the user doesn't exist).

- **deleteMessage(channelName, messageId)**:

* Description: Attempts to delete a message from a chat channel. **Important:** This method relies on the DefaultChatGrape module being loaded and having a deleteMessageFromChannel method exposed on its API.

* Parameters:

- channelName (string): The name of the channel (must be a DefaultChatGrape channel).

- messageId (string): The ID of the message to delete.

* Returns: boolean (indicates if the deletion attempt was made via the module's API; actual success depends on the module's implementation and if the message exists). Returns false if the channel isn't handled by a compatible DefaultChatGrape.

- **getIO()**:

* Description: Returns the global socket.io server instance for advanced use cases (use with caution, direct manipulation can interfere with module operations).

* Returns: object (the socket.io server instance).


## **GrapeAPI**

The GrapeAPI provides methods for Grape Modules (which define core server functionalities and channel types) to interact with the server, manage custom channel types, handle socket events, and persist data. A Grape Module gets its own API instance when its initialize function is called.


### **API Methods**

- **moduleName**:

* Description: The name of the Grape Module this API instance belongs to.

* Returns: string

- **log(message)**:

* Description: Logs a message to the server console, prefixed with the Grape Module's name.

* Parameters:

- message (string): The message to log.

* **getIO()**:

- Description: Returns the global socket.io server instance passed during initialization.

- Returns: object (the socket.io server instance).

* **registerChannelType(config)**:

- Description: Registers a custom channel type handled by this module. This defines how channels of this type behave, their properties, and custom handlers for events like joining or sending messages. If a type name conflicts with an existing one, it will overwrite it (a warning is logged).

- Parameters:

* config (object): Configuration object for the channel type.

- name (string): The internal type name (e.g., 'chat', 'forum'). **Required**.

- displayName (string): User-facing name for the type (e.g., 'Chat Channel', 'Forum Board'). **Required**.

- moduleOwnerName (string): Automatically set to the module's name.

- isPubliclyWritable (boolean, optional): Whether users can write to channels of this type by default (defaults to true). Admins can always write.

- initialData (function | object, optional): Function or object providing the initial data structure for new channels of this type. If a function, it receives (channelName, createdBy). Defaults to an empty object {}.

- charLimit (number, optional): Custom character limit for messages/content in this channel type. If omitted, uses server defaults based on type or the global default.

- icon (string, optional): Default icon character/string for channels of this type displayed in the channel list.

- clientProperties (function, optional): A function (channel, channelName, charLimit) that returns an object of additional properties to send to the client in the channel list for this type. Merged with base properties.

- joinChannelHandler (function, optional): A function (socket, channelData, grapeApi, canWrite, charLimit) called when a user joins a channel of this type. Responsible for sending initial channel content/state to the joining user.

- messageHandler (function, optional): A function (socket, messageData, channelState, grapeApi) called when a user sends a message (sendMessage event) to a channel of this type. Responsible for processing the message/action, updating state, and broadcasting necessary updates.

- _(Other custom handlers can be defined in the config and used internally by the module, but are not directly invoked by the core server)_

* **addSocketListener(eventName, handler)**:

- Description: Registers a handler for a specific socket.io event on _all_ incoming connections. Useful for module-specific global events.

- Parameters:

* eventName (string): The name of the socket.io event.

* handler (function): The function to execute. Receives (socket, ...args, grapeApi) as arguments, where grapeApi is the API instance for _this_ module.

- **getChannelData(channelName)**:

* Description: Gets a deep copy of the data for a specific channel from the server's main channel store.

* Parameters:

- channelName (string): The name of the channel.

* Returns: object | null (a copy of the channel data, or null if the channel doesn't exist).

- **saveChannelData(channelName, data)**:

* Description: Overwrites the _entire_ data object for a specific channel in the main store and triggers a save of _all_ channels to disk. Use with caution; prefer updateChannelField for targeted updates.

* Parameters:

- channelName (string): The name of the channel.

- data (object): The complete new data object for the channel.

* Returns: boolean (true if channel existed and save was attempted, false otherwise).

- **updateChannelField(channelName, fieldPath, value)**:

* Description: Updates a specific field (potentially nested using dot notation like 'settings.moderators') within a channel's data object in the main store and triggers a save of _all_ channels to disk. Creates intermediate objects if they don't exist.

* Parameters:

- channelName (string): The name of the channel.

- fieldPath (string): Dot-separated path to the field within the channel's data.

- value (any): The new value for the field.

* Returns: boolean (true if the channel and its data property existed and save was attempted, false otherwise).

- **getUserData(username)**:

* Description: Gets a deep copy of the data for a specific user from the server's main user store.

* Parameters:

- username (string): The username.

* Returns: object | null (a copy of the user data, or null if the user doesn't exist).

- **broadcastToChannel(channelName, event, data)**:

* Description: Emits a socket.io event to all clients currently in the specified channel's room.

* Parameters:

- channelName (string): The target channel name/room.

- event (string): The event name.

- data (any): The data payload for the event.

* **emitToSocket(socketId, event, data)**:

- Description: Emits a socket.io event directly to a specific client socket.

- Parameters:

* socketId (string): The ID of the target socket.

* event (string): The event name.

* data (any): The data payload for the event.

- **storeModuleData(key, value)**:

* Description: Asynchronously stores JSON-serializable data specific to this Grape Module. Data is saved in chat\_data/grape\_module\_data/\<moduleName>/.

* Parameters:

- key (string): The key (filename without extension).

- value (any): The JSON-serializable data.

* Returns: Promise\<boolean> (true on success, false on error).

- **retrieveModuleData(key)**:

* Description: Asynchronously retrieves previously stored data for this Grape Module.

* Parameters:

- key (string): The key of the data to retrieve.

* Returns: Promise\<any | null> (the retrieved data, or null if not found or on error).

- **getUserProfile(username)**:

* Description: Retrieves a formatted user profile object, suitable for sending to clients (e.g., in messages). Includes defaults for missing information like avatar details.

* Parameters:

- username (string): The username.

* Returns: object ({ username, avatarText, avatarUrl, avatarBg, id, roles }). Returns default values if the user doesn't exist.

- **sendHtmlToSocket(socketId, targetElementSelector, htmlString)**:

* Description: Sends an HTML string to a specific client via the renderHtml event, instructing the client-side code to render the HTML within the element matching the CSS selector.

* Parameters:

- socketId (string): The ID of the target socket.

- targetElementSelector (string): A CSS selector for the target element on the client.

- htmlString (string): The HTML content to render.

* **broadcastHtmlToChannel(channelName, targetElementSelector, htmlString)**:

- Description: Sends an HTML string to all clients in a channel via the renderHtml event, instructing their client-side code to render the HTML within the element matching the CSS selector.

- Parameters:

* channelName (string): The target channel name/room.

* targetElementSelector (string): A CSS selector for the target element on the clients.

* htmlString (string): The HTML content to render.

- **ensureMessageId(message)**:

* Description: Utility function that checks if a message object has a messageId property. If not, it adds one using uuidv4() and returns the modified message object.

* Parameters:

- message (object): The message object to check/modify.

* Returns: object (The message object, guaranteed to have a messageId).

- **saveAllChannels()**:

* Description: Manually triggers a save of the entire channels data structure to channels.json. Use sparingly, as updates often trigger saves automatically.

- **getAllChannels()**:

* Description: Gets a deep copy of the _entire_ channels data structure from the server's main store.

* Returns: object (A copy of all channel data).

- **getAllUsers()**:

* Description: Gets a deep copy of the _entire_ users data structure from the server's main store.

* Returns: object (A copy of all user data).

- **getChannelTypeConfig(typeName)**:

* Description: Retrieves a copy of the registered configuration object for a specific channel type.

* Parameters:

- typeName (string): The internal name of the channel type.

* Returns: object | null (A copy of the type configuration, or null if the type isn't registered).

- **createSnippet(text, maxLength = THREAD\_SNIPPET\_LENGTH)**:

* Description: Utility function to create a short text snippet, truncating with "..." if it exceeds the maximum length.

* Parameters:

- text (string): The input text.

- maxLength (number, optional): The maximum length before truncation (defaults to the server's THREAD\_SNIPPET\_LENGTH).

* Returns: string (The potentially truncated snippet).
