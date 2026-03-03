# ClaverIT Chat WebSocket Protocol Documentation

WebSocket real-time messaging protocol for the chat-backend service.

**Production URL:** `wss://claverit-chat-backend.onrender.com`

---

## Connection

### Establishing Connection

```
wss://claverit-chat-backend.onrender.com?token=<JWT_TOKEN>
```

**Query Parameters:**
| Parameter | Required | Description |
|-----------|----------|-------------|
| `token` | Yes | JWT token from main-backend authentication |

### Connection Flow

1. Client connects with JWT token as query parameter
2. Server validates token and extracts user identity
3. On success: connection established, `connection_ack` event emitted
4. On failure: connection closed with error code

### Error Codes

| Code | Description |
|------|-------------|
| 1000 | Normal closure |
| 1008 | Policy violation (invalid/expired token) |
| 1011 | Server error |

---

## Authentication

All WebSocket connections require a valid JWT token. The token is passed as a query parameter during the initial handshake.

```javascript
const socket = new WebSocket(
  `wss://claverit-chat-backend.onrender.com?token=${accessToken}`
);
```

---

## Events (Client → Server)

### `join_room`

Join a conversation room to receive real-time messages.

**Payload:**
```json
{
  "event": "join_room",
  "data": {
    "conversationId": "string"
  }
}
```

**Response Event:** `room_joined`

---

### `leave_room`

Leave a conversation room.

**Payload:**
```json
{
  "event": "leave_room",
  "data": {
    "conversationId": "string"
  }
}
```

**Response Event:** `room_left`

---

### `send_message`

Send a text message to a conversation.

**Payload:**
```json
{
  "event": "send_message",
  "data": {
    "conversationId": "string",
    "content": "string",
    "clientMessageId": "string (optional, for deduplication)"
  }
}
```

**Response Event:** `message_sent` (to sender), `new_message` (to room)

---

### `typing_start`

Indicate user started typing.

**Payload:**
```json
{
  "event": "typing_start",
  "data": {
    "conversationId": "string"
  }
}
```

**Broadcast Event:** `user_typing` (to other room members)

---

### `typing_stop`

Indicate user stopped typing.

**Payload:**
```json
{
  "event": "typing_stop",
  "data": {
    "conversationId": "string"
  }
}
```

**Broadcast Event:** `user_stopped_typing` (to other room members)

---

### `mark_read`

Mark messages as read up to a certain point.

**Payload:**
```json
{
  "event": "mark_read",
  "data": {
    "conversationId": "string",
    "lastReadMessageId": "string (optional)"
  }
}
```

**Response Event:** `read_receipt_updated`

---

### `ping`

Keep-alive ping (server responds with pong).

**Payload:**
```json
{
  "event": "ping"
}
```

**Response Event:** `pong`

---

## Events (Server → Client)

### `connection_ack`

Emitted when connection is successfully established.

**Payload:**
```json
{
  "event": "connection_ack",
  "data": {
    "userId": "string",
    "connectedAt": "ISO8601 timestamp"
  }
}
```

---

### `room_joined`

Confirmation of successfully joining a room.

**Payload:**
```json
{
  "event": "room_joined",
  "data": {
    "conversationId": "string",
    "participantCount": "number"
  }
}
```

---

### `room_left`

Confirmation of leaving a room.

**Payload:**
```json
{
  "event": "room_left",
  "data": {
    "conversationId": "string"
  }
}
```

---

### `new_message`

A new message was received in a joined room.

**Payload:**
```json
{
  "event": "new_message",
  "data": {
    "id": "string",
    "conversationId": "string",
    "senderId": "string",
    "senderName": "string",
    "senderProfileImage": "string | null",
    "content": "string",
    "messageType": "text | image | file | system",
    "media": [
      {
        "id": "string",
        "url": "string",
        "type": "image | video | audio | file",
        "filename": "string",
        "size": "number",
        "mimeType": "string"
      }
    ],
    "clientMessageId": "string | null",
    "createdAt": "ISO8601 timestamp"
  }
}
```

---

### `message_sent`

Confirmation that your message was sent successfully.

**Payload:**
```json
{
  "event": "message_sent",
  "data": {
    "id": "string",
    "conversationId": "string",
    "clientMessageId": "string | null",
    "createdAt": "ISO8601 timestamp"
  }
}
```

---

### `message_updated`

A message was edited.

**Payload:**
```json
{
  "event": "message_updated",
  "data": {
    "id": "string",
    "conversationId": "string",
    "content": "string",
    "updatedAt": "ISO8601 timestamp"
  }
}
```

---

### `message_deleted`

A message was deleted.

**Payload:**
```json
{
  "event": "message_deleted",
  "data": {
    "id": "string",
    "conversationId": "string",
    "deletedAt": "ISO8601 timestamp"
  }
}
```

---

### `user_typing`

Another user started typing in a conversation.

**Payload:**
```json
{
  "event": "user_typing",
  "data": {
    "conversationId": "string",
    "userId": "string",
    "userName": "string"
  }
}
```

---

### `user_stopped_typing`

Another user stopped typing.

**Payload:**
```json
{
  "event": "user_stopped_typing",
  "data": {
    "conversationId": "string",
    "userId": "string"
  }
}
```

---

### `read_receipt_updated`

Read status was updated for a conversation.

**Payload:**
```json
{
  "event": "read_receipt_updated",
  "data": {
    "conversationId": "string",
    "userId": "string",
    "lastReadMessageId": "string",
    "readAt": "ISO8601 timestamp"
  }
}
```

---

### `user_online`

A user in your conversations came online.

**Payload:**
```json
{
  "event": "user_online",
  "data": {
    "userId": "string"
  }
}
```

---

### `user_offline`

A user in your conversations went offline.

**Payload:**
```json
{
  "event": "user_offline",
  "data": {
    "userId": "string",
    "lastSeen": "ISO8601 timestamp"
  }
}
```

---

### `conversation_updated`

Conversation metadata was updated (name, participants, etc.).

**Payload:**
```json
{
  "event": "conversation_updated",
  "data": {
    "conversationId": "string",
    "changes": {
      "name": "string | null",
      "participantsAdded": ["string"],
      "participantsRemoved": ["string"]
    },
    "updatedAt": "ISO8601 timestamp"
  }
}
```

---

### `friend_request_received`

A new friend request was received.

**Payload:**
```json
{
  "event": "friend_request_received",
  "data": {
    "requestId": "string",
    "senderId": "string",
    "senderName": "string",
    "senderProfileImage": "string | null",
    "createdAt": "ISO8601 timestamp"
  }
}
```

---

### `friend_request_accepted`

Your friend request was accepted.

**Payload:**
```json
{
  "event": "friend_request_accepted",
  "data": {
    "requestId": "string",
    "friendId": "string",
    "friendName": "string",
    "acceptedAt": "ISO8601 timestamp"
  }
}
```

---

### `error`

An error occurred processing a request.

**Payload:**
```json
{
  "event": "error",
  "data": {
    "code": "string",
    "message": "string",
    "originalEvent": "string | null"
  }
}
```

**Error Codes:**
| Code | Description |
|------|-------------|
| `AUTH_FAILED` | Authentication failed |
| `ROOM_NOT_FOUND` | Conversation not found |
| `NOT_MEMBER` | User is not a member of the conversation |
| `MESSAGE_FAILED` | Failed to send message |
| `RATE_LIMITED` | Too many requests |
| `INVALID_PAYLOAD` | Invalid event payload |

---

## Example Client Implementation

```javascript
class ChatWebSocket {
  constructor(token) {
    this.token = token;
    this.socket = null;
    this.reconnectAttempts = 0;
    this.maxReconnectAttempts = 5;
  }

  connect() {
    this.socket = new WebSocket(
      `wss://claverit-chat-backend.onrender.com?token=${this.token}`
    );

    this.socket.onopen = () => {
      console.log('WebSocket connected');
      this.reconnectAttempts = 0;
    };

    this.socket.onmessage = (event) => {
      const { event: eventName, data } = JSON.parse(event.data);
      this.handleEvent(eventName, data);
    };

    this.socket.onclose = (event) => {
      if (event.code !== 1000 && this.reconnectAttempts < this.maxReconnectAttempts) {
        this.reconnectAttempts++;
        setTimeout(() => this.connect(), 1000 * this.reconnectAttempts);
      }
    };

    this.socket.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  }

  send(event, data) {
    if (this.socket?.readyState === WebSocket.OPEN) {
      this.socket.send(JSON.stringify({ event, data }));
    }
  }

  joinRoom(conversationId) {
    this.send('join_room', { conversationId });
  }

  leaveRoom(conversationId) {
    this.send('leave_room', { conversationId });
  }

  sendMessage(conversationId, content, clientMessageId = null) {
    this.send('send_message', { conversationId, content, clientMessageId });
  }

  startTyping(conversationId) {
    this.send('typing_start', { conversationId });
  }

  stopTyping(conversationId) {
    this.send('typing_stop', { conversationId });
  }

  markRead(conversationId, lastReadMessageId = null) {
    this.send('mark_read', { conversationId, lastReadMessageId });
  }

  handleEvent(eventName, data) {
    // Override in subclass or set up event listeners
    console.log(`Received ${eventName}:`, data);
  }

  disconnect() {
    if (this.socket) {
      this.socket.close(1000, 'Client disconnect');
      this.socket = null;
    }
  }
}
```

---

## Rate Limiting

- **Messages:** 30 messages per minute per conversation
- **Typing indicators:** 10 per minute per conversation
- **Room joins:** 20 per minute

Exceeding limits will result in an `error` event with code `RATE_LIMITED`.

---

## Reconnection Strategy

1. Implement exponential backoff (1s, 2s, 4s, 8s, 16s)
2. Max 5 reconnection attempts
3. On reconnection, re-join all previously joined rooms
4. Request missed messages via REST API if needed

---

## Best Practices

1. **Always handle `error` events** - Log and display appropriate user feedback
2. **Use `clientMessageId`** - For message deduplication and optimistic UI updates
3. **Implement typing debounce** - Send `typing_start` on first keystroke, `typing_stop` after 2s of inactivity
4. **Keep connection alive** - Send `ping` every 30 seconds if no other activity
5. **Handle offline gracefully** - Queue messages locally, sync via REST when reconnected
