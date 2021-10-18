# SocketIO

To connect to a socket server, you need to add the '_' parameter with the access_token value.

```log
ws://185.177.105.151:1998?_=0013bbbcf2f21b30709ea4c186...
```
- - - 

##### # Server-side Events:
+ **connect**
    - We receive it upon successful connection.
    - Here we also get and save the client's ID.
+ **signal**
    - We get this when something is created, updated or deleted.
    - Here we have 2 arguments. **SignalType** and **SignalData**
    - The **SignalData** is a json data: **{ "data": any, "error": true or false }**
    + The **SignalType** is a number. Below are the meanings of the signals.
        + **SIGNAL_PROFILE = 0**
            - You will receive this signal after successful authentication on the server.
        + **SIGNAL_CONNECTED = 1**
            - This is triggered when clients connects to the server.
        + **SIGNAL_DISCONNECTED = 2**
            - This is triggered when clients disconnects from the server.
        + **SIGNAL_R_JOINED = 3**
            - This is triggered when clients join to the room.
        + **SIGNAL_R_LEAVED = 4**
            - This is triggered when clients left from the room.
        + **SIGNAL_R_INSERT = 5**
            - This is triggered when clients create a new room.
        + **SIGNAL_R_UPDATE = 6**
            - This is triggered when clients are editing existing rooms.
        + **SIGNAL_R_DELETE = 7**
            - This is triggered when clients are deleting existing rooms.
        + **SIGNAL_M_INSERT = 8**
            - This is triggered when clients send a new message.
        + **SIGNAL_M_UPDATE = 9**
            - This is triggered when clients are editing existing messages.
        + **SIGNAL_M_DELETE = 10**
            - This is triggered when clients are deleting existing messages.
        + **SIGNAL_M_TYPING = 11**
            - This is triggered when clients are typing.
        + **SIGNAL_M_RECEIVED = 12**
            - This is triggered when clients are received or viewed messages.
        + **SIGNAL_NOTIFICATION = 13**
            - This is triggered when we receive notifications.
        + **SIGNAL_R_UPDATE_MEMBERS = 14**
            - This is triggered when clients are creating, editing or deleting members in existing rooms.
        + **SIGNAL_SOS = 15**
            * **Doesn't work yet.*
        + **SIGNAL_M_LIKED = 16**
            - This is triggered when clients are liked existing messages.
+ **connect_error**
    - This is received if an error occurred while connecting to the server.
+ **disconnect**
    - We get this if we disconnected from the server.

- - - 

##### # Client-side Emitter:
+ **SIGNAL_M_TYPING**
    - 1 argument is: 11
    + 2 argument is: { "typing": true }
        - typing = true // when you typing
        - typing = false // when you don't typing
+ **SIGNAL_M_RECEIVED**
    - 1 argument is: 12
    + 2 argument is: { "received_room_id": 1, "received_message_id": 2, "received_type": 1 }
        - received_type = 1 // message received but not read.
        - received_type = 2 // message received and read.
+ **SIGNAL_M_LIKED**
    - 1 argument is: 16
    + 2 argument is: { "like_room_id": 1, "like_message_id": 2, "like_is_liked": 1 }
        - like_is_liked = -1 // negative 👎
        - like_is_liked = 0  // neutral 😶
        - like_is_liked = 1  // positive 👍

- - - 


#### # Example using in Kotlin
```kotlin
import io.socket.client.IO
import io.socket.client.Socket
import org.json.JSONObject

class SignalerSocketClient() {
    companion object {
        private const val SOCKET_BASE_URL: String = "ws://185.177.105.151:1998"
        private const val SIGNAL_NAME: String = "signal"
        const val SIGNAL_PROFILE = 0
        const val SIGNAL_CONNECTED = 1
        const val SIGNAL_DISCONNECTED = 2
        const val SIGNAL_R_JOINED = 3
        const val SIGNAL_R_LEAVED = 4
        const val SIGNAL_R_INSERT = 5
        const val SIGNAL_R_UPDATE = 6
        const val SIGNAL_R_DELETE = 7
        const val SIGNAL_M_INSERT = 8
        const val SIGNAL_M_UPDATE = 9
        const val SIGNAL_M_DELETE = 10
        const val SIGNAL_M_TYPING = 11
        const val SIGNAL_M_RECEIVED = 12
        const val SIGNAL_NOTIFICATION = 13
        const val SIGNAL_R_UPDATE_MEMBERS = 14
        const val SIGNAL_SOS = 15
        const val SIGNAL_M_LIKED = 16
    }

    private val options = IO.Options()
    private var mSocket: Socket? = null
    private var accessToken: String? = "0013bbbcf2f21b30709ea4c186..."

    init {
        options.forceNew = true
        options.reconnection = true
        options.reconnectionAttempts = 10
        options.reconnectionDelay = 100
        options.query = "_=$accessToken"
    }

    fun connect() {
        mSocket = IO.socket(SOCKET_BASE_URL, options)

        mSocket?.on(Socket.EVENT_CONNECT) { }
        mSocket?.on(Socket.EVENT_CONNECT_ERROR) { args: Array<Any>? -> }
        mSocket?.on(Socket.EVENT_DISCONNECT) { args: Array<Any>? -> }
        mSocket?.on(SIGNAL_NAME) { (signal_type: Any, signal_data: Any) ->
            if (signal_type is Int && signal_data is JSONObject) {
                when (signal_type) {
                    SIGNAL_PROFILE          -> { /* ...code... */ }
                    SIGNAL_CONNECTED        -> { /* ...code... */ }
                    SIGNAL_DISCONNECTED     -> { /* ...code... */ }
                    SIGNAL_R_JOINED         -> { /* ...code... */ }
                    SIGNAL_R_LEAVED         -> { /* ...code... */ }
                    SIGNAL_R_INSERT         -> { /* ...code... */ }
                    SIGNAL_R_UPDATE         -> { /* ...code... */ }
                    SIGNAL_R_DELETE         -> { /* ...code... */ }
                    SIGNAL_M_INSERT         -> { /* ...code... */ }
                    SIGNAL_M_UPDATE         -> { /* ...code... */ }
                    SIGNAL_M_DELETE         -> { /* ...code... */ }
                    SIGNAL_M_TYPING         -> { /* ...code... */ }
                    SIGNAL_M_RECEIVED       -> { /* ...code... */ }
                    SIGNAL_NOTIFICATION     -> { /* ...code... */ }
                    SIGNAL_R_UPDATE_MEMBERS -> { /* ...code... */ }
                    SIGNAL_SOS              -> { /* ...code... */ }
                    SIGNAL_M_LIKED          -> { /* ...code... */ }
                }
            }
        }

        mSocket?.connect()
    }

    fun close() {
        if (mSocket != null) {
            mSocket?.close()
        }
    }

    private fun signal(type: Int, signalData: JSONObject) {
        if (mSocket != null) {
            mSocket?.emit(SIGNAL_NAME, type, signalData)
        }
    }

    fun sendTyping(typing: Bool) {
        val signalData = JSONObject()
        signalData.put("typing", typing)
        signal(SIGNAL_M_TYPING, signalData);
    }

    fun sendReceivedMessage(room_id: Int, message_id: Int, type: Int) {
        val signalData = JSONObject()
        signalData.put("received_room_id", room_id)
        signalData.put("received_message_id", message_id)
        signalData.put("received_type", type)
        signal(SIGNAL_M_RECEIVED, signalData);
    }

    fun sendLikedMessage(room_id: Int, message_id: Int, is_liked: Int) {
        val signalData = JSONObject()
        signalData.put("like_room_id", room_id)
        signalData.put("like_message_id", message_id)
        signalData.put("like_is_liked", is_liked)
        signal(SIGNAL_M_LIKED, signalData);
    }
}

```

- - - 

# HTTP APIs
###### PostmanCollection: [Download](https://raw.githubusercontent.com/MARK-46/universe_chat_doc/main/%23CHAT.postman_collection.json)

For make request you need to add three headers.
```log
1) Authorization: 0013bbbcf2f21b30709ea4c186... # This is the oauth access_token
2) _: ZkJbjUa6n3CgkuGXAAAB                      # This is the ID of the socket client
3) Accept-Language: EN                          # This is the response messages language
```
- - - 

#### # Get a list of available rooms.
Query Parameters:
+ **type** - No Required. 
    - 1 - Private Group(OneToMany)
    - 2 - Private Chat(OneToOne)
    - 3 - Public Group(OneToMany)
```log
Method: GET  | Url: http://185.177.105.151:1998/api/rooms/list?type=1
```
- - - 

#### # Make this request to subscribe a global public room.
This is a request to subscribe to a global room to receive action notifications.
Parameters:
+ **room_id** - Required/Number.
```log
Method: GET  | Url: http://185.177.105.151:1998/api/rooms/:room_id/subscribe
```
- - - 

#### # Make this request to join a room.
Parameters:
+ **room_id** - Required/Number.
```log
Method: GET  | Url: http://185.177.105.151:1998/api/rooms/:room_id/join
```
- - - 

#### # Make this request before joining a room to leave the previous room.
Parameters:
+ **room_id** - Required/Number.
```log
Method: GET  | Url: http://185.177.105.151:1998/api/rooms/:room_id/leave
```
- - - 

#### # Get room members.
Parameters:
+ **room_id** - Required/Number.
```log
Method: GET  | Url: http://185.177.105.151:1998/api/rooms/:room_id/members/list
```
- - - 

#### # Get room messages by parameters limit and skip.
Parameters:
+ **room_id** - Required/Number.

Query Parameters:
+ **limit** - No Required/Number. Used to specify the maximum number of results to be returned
+ **skip** - No Required/Number. Gets or sets the number of search results to skip.
```log
Method: GET  | Url: http://185.177.105.151:1998/api/rooms/:room_id/messages/list?limit=10&skip=0
```
- - - 

#### # Get a list of available users to create a room.
Query Parameters:
+ **joint_room_type** - Required. To show a list of shared rooms.
    - 1 - Private Group(OneToMany)
    - 2 - Private Chat(OneToOne)
    - 3 - Public Group(OneToMany)
+ **limit** - No Required/Number. Used to specify the maximum number of results to be returned
+ **skip** - No Required/Number. Gets or sets the number of search results to skip.
```log
Method: GET  | Url: http://185.177.105.151:1998/api/friends/list?joint_room_type=1&limit=10&skip=0
```
- - - 

#### # To add a new or edit an existing room (if has access).
Parameters:
+ **room_id** - Required/Number.

FormData Fields:
+ **room_name** - Required/String.
+ **room_image** - Required/Binary File or Image Full Path.
+ **room_type** - Required/Number. 
    - 1 - Private Group(OneToMany)
    - 2 - Private Chat(OneToOne)
    - 3 - Public Group(OneToMany)
+ **room_owner[0]** - Required/MultipleNumber. User ID
+ **room_member[0]** - Required/MultipleNumber. User ID
```log
Method: POST | Url: http://185.177.105.151:1998/api/rooms/create
Method: POST | Url: http://185.177.105.151:1998/api/rooms/:room_id/update
Exmple Body:
    room_name:MyRoom
    room_type:1
    
    room_owner[0]:4532
    
    room_member[0]:75137
    room_member[1]:25433

    room_image:<binary_file>
    or
    room_image:http://136.244.117.119:88/upload/images/users/profiles/admins/mark46.png
```
- - - 

#### # To add a new or edit an existing message (if has access).
Parameters:
+ **room_id** - Required/Number.
+ **message_id** - Required/Number.

FormData Fields:
+ **message_content** - Required/String. The content of the message is here
+ **message_type** - Required/Number. 
    - 1 - Text message with replies or files.
    - 2 - Message when only image|audio|video.
    - 3 - Event message
+ **message_mention_options** - Required/String. Here is JSON string data
+ **message_replies[0]** - Required/MultipleNumber. ID of the messages you are reply to. 
+ **message_files[0]** - Required/Binary File
+ **message_deleted_files[0]** - No Required/MultipleNumber. ID of the files to be deleted.
```log
Method: POST | Url: http://185.177.105.151:1998/api/rooms/:room_id/messages/send
Method: POST | Url: http://185.177.105.151:1998/api/rooms/:room_id/messages/:message_id/update
Exmple Body:
    message_content:Hello Chat!
    message_type:1
    message_mention_options:[{"test": "text"}]
    message_replies[0]:431
    message_replies[1]:432
    message_files[0]:<binary_file>
    message_files[1]:<binary_file>
    message_deleted_files[0]:1
    message_deleted_files[1]:2
```
- - - 

#### # To delete a room or room message (if has access).
Parameters:
+ **room_id** - Required/Number.
+ **message_id** - Required/Number.
```log
Method: POST | Url: http://185.177.105.151:1998/api/rooms/:room_id/delete
Method: POST | Url: http://185.177.105.151:1998/api/rooms/:room_id/messages/:message_id/delete
```
