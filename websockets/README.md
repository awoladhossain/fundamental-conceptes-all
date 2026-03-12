# 🔌 WebSockets Learning Notes

## 📂 Topics Covered

| File | Description | Status |
|------|-------------|--------|
| [WebSocket](websocket.md) | WebSocket Fundamentals | ✅ Done |

## 🎯 Quick Reference

### HTTP vs WebSocket
| Feature | HTTP | WebSocket |
|---------|------|-----------|
| Connection | Request-Response | Persistent |
| Direction | Client-initiated | Bidirectional |
| Overhead | Headers each time | Low after handshake |
| Use Case | REST APIs | Real-time apps |

### WebSocket Lifecycle
```
1. Client: HTTP Upgrade Request
2. Server: 101 Switching Protocols
3. Both: Full-duplex communication
4. Either: Close connection
```

### Client Example (JavaScript)
```javascript
const ws = new WebSocket('ws://localhost:8080');

ws.onopen = () => {
  console.log('Connected');
  ws.send('Hello Server!');
};

ws.onmessage = (event) => {
  console.log('Received:', event.data);
};

ws.onclose = () => {
  console.log('Disconnected');
};

ws.onerror = (error) => {
  console.error('Error:', error);
};
```

### Server Example (Node.js)
```javascript
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
  ws.on('message', (message) => {
    console.log('Received:', message);
    ws.send('Echo: ' + message);
  });
});
```

### Use Cases
- 💬 Chat applications
- 🎮 Online gaming
- 📊 Real-time dashboards
- 📈 Stock tickers
- 🔔 Notifications
- 👥 Collaborative editing

## 📚 Resources
- [MDN WebSocket API](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)
- [Socket.io](https://socket.io/)

---
*Last Updated: March 2026*
