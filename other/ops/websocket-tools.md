
## 🔌 WebSocket Tools: `wscat` & `websocket-echo-server`

This README provides a quick reference and usage guide for two helpful WebSocket tools:

- [`wscat`](https://github.com/websockets/wscat): A command-line WebSocket client.
- [`websocket-echo-server`](https://github.com/websockets/websocket-echo-server): A simple WebSocket server that echoes messages back to clients.

---

### 📦 Installation

Install both tools globally using `npm`:

```bash
npm install -g wscat websocket-echo-server
```

---

### 🧪 1. Using `websocket-echo-server`

#### 🔄 Start Echo Server

```bash
websocket-echo-server -p 8080
```

- Default port is `8080`
- Can be changed with `-p <port>`

#### 🔍 Output

The server logs all messages received and echoes them back to all connected clients.

---

### 🧑‍💻 2. Using `wscat` (WebSocket Client)

#### 📡 Connect to a WebSocket Server

```bash
wscat -c ws://localhost:8080
```

Once connected, you can type messages — they will be sent to the server and echoed back.

#### ⚙️ Additional Options

- Send JSON:
  ```bash
  echo '{"event":"ping"}' | wscat -c ws://localhost:8080
  ```
- Secure WebSocket (`wss://`) is also supported.

---

### 🧪 Local Test Flow

1. Start the echo server:

   ```bash
   websocket-echo-server -p 8080
   ```

2. In a separate terminal, connect using wscat:

   ```bash
   wscat -c ws://localhost:8080
   ```

3. Type a message and observe the echo.
   ```bash
   wscat -c wss://websocket-echo.com
   Connected (press CTRL+C to quit)
   > Hi
   < Hi
   > {"event":"ping"}
   < {"event":"ping"}
   ```

---

### 📚 Useful Links

- [wscat GitHub](https://github.com/websockets/wscat)
- [websocket-echo-server GitHub](https://github.com/websockets/websocket-echo-server)
- [WebSocket Protocol RFC](https://datatracker.ietf.org/doc/html/rfc6455)

---

> 🧠 Tip: Use these tools to debug or prototype WebSocket-based apps and APIs quickly.
