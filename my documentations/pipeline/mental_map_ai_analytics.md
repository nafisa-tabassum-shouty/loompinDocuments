# 🗺️ Mental Map: The "Life of a Query" (Guided Trace)

To understand how Loompin works without reading thousands of lines, follow this specific **AI Analytics** flow. This shows you how a single question from a user travels through 6 different files.

---

### **1. The Gateway**
**File: [apiRoutes.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/routes/apiRoutes.js#L360)**
- **What happens**: The system defines the route `POST /analytics/ask`.
- **The Connection**: It points to `analyticsController.askAI`.

### **2. The Coordinator**
**File: [analyticsController.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/controllers/analyticsController.js#L263-290)**
- **What happens**: The controller receives your question (e.g., "What is the NPS trend?").
- **The Connection**: It calls `orchestrator.dispatch(...)` and sends the task to the **'analytics'** squad.

### **3. The Expert Brain**
**File: [DataAnalystAgent.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/agents/squads/DataAnalystAgent.js#L28-31)**
- **What happens**: The Data Analyst Agent "wakes up." It realizes it needs to talk to the database to answer you.
- **The Connection**: It calls the `naturalQueryEngine.ask(...)` method.

### **4. The Translator**
**File: [NaturalQueryEngine.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/NaturalQueryEngine.js#L35-120)**
- **What happens**: This is the "Magic" part. It uses Gemini to turn your English question into a **SQL Query**.
- **The Connection**: It runs a safety check (`_isReadOnly`) and then calls `query(...)` to talk to the actual database.

### **5. The Memory**
**File: [db.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loombot_VoC/backend/config/db.js)**
- **What happens**: The raw SQL query is executed against your PostgreSQL database.
- **The Connection**: It returns the raw data rows back up to the engine.

### **6. The Visualization Cleanup**
**Back in: [DataAnalystAgent.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/agents/squads/DataAnalystAgent.js#L78-101)**
- **What happens**: The agent looks at the data rows. If it sees dates and numbers, it says, "A Line Chart would look great here!"
- **The Connection**: It returns the data + a `chartConfig` back to the UI.

---

### **Summary: The Chain**
`Route` $\rightarrow$ `Controller` $\rightarrow$ `Orchestrator` $\rightarrow$ `Agent` $\rightarrow$ `Engine` $\rightarrow$ `Database`

By following this **Trace**, you have just learned the architecture of 6 major files in less than 5 minutes!
