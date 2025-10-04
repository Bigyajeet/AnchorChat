#  AnchorChat: Scalable, Reliable Real-Time Chat Application

## Overview

**AnchorChat** is a high-performance, real-time chat application built using **Node.js, Express, and Socket.IO**. It is architected for **horizontal scalability** using Node.js clustering and features robust **message persistence** via SQLite, ensuring reliable delivery and temporary connection recovery.

This application is designed to handle a large volume of concurrent users while guaranteeing that no messages are lost, even during brief network interruptions.

##  Core Features

| Feature | Technology/Concept | Benefit |
| :--- | :--- | :--- |
| **High Scalability** | Node.js Cluster & `@socket.io/cluster-adapter` | Distributes load across all available CPU cores, allowing the application to scale horizontally and maintain performance under heavy load. |
| **Message Persistence** | SQLite (`chat.db`) | All messages are stored in a local database file, ensuring data integrity across server restarts. |
| **Connection Recovery (QoS)** | `connectionStateRecovery` | Upon temporary disconnection and reconnection, the server automatically resends any missed messages using the last known **server offset** (message ID). |
| **Broadcast & Targeting** | `io.emit()` & `io.to()` | Enables sending messages to **all** connected clients or a **subset** of clients (e.g., chat rooms). |
| **Duplicate Prevention** | `client_offset` (Unique Constraint) | Prevents the same message from being inserted into the database multiple times during client-side network retries. |

-----

## ⚙️ Setup and Installation

### Prerequisites

  * Node.js (v18+)
  * npm or Yarn

### Steps

1.  **Clone the Repository:**

    ```bash
    git clone [your-repo-url]
    cd AnchorChat
    ```

2.  **Install Dependencies:**

    ```bash
    npm install
    ```

    This will install `express`, `socket.io`, `sqlite`, `sqlite3`, and `@socket.io/cluster-adapter`.

3.  **Run the Server:**
    The application automatically starts in cluster mode, utilizing all available CPU cores and listening on the shared port **3000**.

    ```bash
    node index.js
    ```

    You will see output indicating that the primary thread is running and workers have been successfully forked.

4.  **Access the Chat:**
    Open your web browser (and multiple tabs/windows) and navigate to:

    ```
    http://localhost:3000
    ```

-----

## 📝 Code Overview

### Server (`index.js`)

  * **Clustering:** The server checks `cluster.isPrimary` and forks workers, setting up the Socket.IO adapter (`setupPrimary()` and `adapter: createAdapter()`).
  * **Database Init:** It initializes the `chat.db` SQLite file and ensures the `messages` table (with `id`, `content`, and `client_offset` columns) exists before listening for connections.
  * **Message Handler:**
    ```javascript
    socket.on('chat message', async (msg, clientOffset, callback) => {
        // 1. Attempts to INSERT message with clientOffset (to prevent duplicates)
        // 2. If successful, broadcasts io.emit('chat message', msg, result.lastID)
        // 3. Acknowledges the client via callback()
    });
    ```
  * **Recovery Logic:** The server uses `socket.handshake.auth.serverOffset` to query the database for missed messages and `socket.emit` them directly back to the recovering client only.

### Client (`index.html`)

  * **Sending:** The client uses `socket.emit('chat message', message, clientOffset, callback)` to send messages with an acknowledgment request.
  * **Receiving:** The client uses `socket.on('chat message', (msg, serverOffset) => { ... });` to listen for new and recovered messages and display them.

-----

## 🌐 Extensibility

To extend AnchorChat into a multi-room application, you would use the following Socket.IO methods:

  * **Joining a Room:** `socket.join('general');`
  * **Targeted Broadcast:** `io.to('general').emit('event', data);`
  * **Targeted Private Message:** `socket.to('another-socket-id').emit('private', data);`

If you are interested in making your GitHub profile more engaging, check out this tutorial on [Creating a Killer GitHub Profile Readme 2023](https://www.youtube.com/watch?v=eHaXw8Bd_ms).
http://googleusercontent.com/youtube_content/0
