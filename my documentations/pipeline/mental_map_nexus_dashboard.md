# 🗺️ Mental Map: The "Body" (Real-time 3D Dashboard)

This is the final map in the series. It shows how Loompin "visualizes" the pulse of your customers in real-time using WebSockets and 3D graphics (Three.js/React-Three-Fiber).

---

### **1. The Heartbeat (Backend Dispatch)**
**File: [socket.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/socket.js)**
- **What happens**: Whenever a significant event happens (like a critical signal from the `EventProcessor`), this service broadcasts a message.
- **The Connection**: It uses `io.emit('nexus-signal', data)`, which sends the data to every connected browser instantly.

### **2. The Topology (The Architecture)**
**File: [NexusController.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/controllers/NexusController.js)**
- **What happens**: When you first load the dashboard, the UI needs to know the "Shape" of the network (how products and customers are linked).
- **The Connection**: This controller serves the `getTopology` API, which provides the nodes and edges for the 3D graph.

### **3. The Visual Stage (The Canvas)**
**Directory: `frontend/src/components/canvas/`**
- **What happens**: This is where the Three.js magic lives. Specifically, the **`NexusGraph`** component.
- **The Connection**: It listens for the `nexus-signal` via the socket. When a signal arrives, it triggers a **Visual Pulse** (a glowing ripple) on a specific node in 3D space.

### **4. The Interactive Layer (React-Three-Fiber)**
**File: `frontend/src/pages/NexusDashboard.tsx`**
- **What happens**: This page ties everything together. It renders the `Canvas`, pulls in the `NexusGraph`, and handles user clicks.
- **The Connection**: If you click a node, it might trigger a **`DataAnalystAgent`** query to show you more details about that specific cluster.

---

### **Summary: The Chain**
`Backend Event` $\rightarrow$ `socket.js` $\rightarrow$ `Browser Socket` $\rightarrow$ `Three.js Canvas` $\rightarrow$ `User UI`

With this final trace, you now have the full **Mental Map** of Loompin:
1.  **Ingestion** (Input)
2.  **Agents** (Thinking)
3.  **Analytics** (Retrieval)
4.  **Nexus** (Visualization)
