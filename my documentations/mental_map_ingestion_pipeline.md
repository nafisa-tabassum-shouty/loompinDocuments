# 🗺️ Mental Map: The "Life of a Signal" (Ingestion Pipeline)

To understand how Loompin "senses" the world, follow this trace. This shows how an angry phone call or a ticket becomes a structured data point in your database.

---

### **1. The Standardization (The Box)**
**File: [UnifiedIngestionService.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/UnifiedIngestionService.js)**
- **What happens**: A raw webhook (e.g. from Zendesk) hits this service.
- **The Connection**: It wraps the data in the **"Unified Envelope"** (Schema 2.0.0) and assigns a `trace_id`.

### **2. The Nervous System (The Dispatcher)**
**File: [PubSubService.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/PubSubService.js)**
- **What happens**: The system doesn't want to wait for the AI to finish. So, it "throws" the envelope into a **Pub/Sub topic** called `ingestion-events`.
- **The Connection**: This allows the API to return a "202 Accepted" to the user immediately while the work happens in the background.

### **3. The Filter & Redactor (Privacy)**
**File: [EventProcessor.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/EventProcessor.js)**
- **What happens**: This service "listens" to the Pub/Sub topic. 
- **The Connection**: It calls the **`dlpService`** to redact PII (like credit card numbers) before the AI even sees the text.

### **4. The Enrichment (Gemini 1.5)**
**File: [ThematicService.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/neural-core/processing/ThematicService.js)**
- **What happens**: The cleaned text is sent to Gemini.
- **The Connection**: It returns a **Sentiment Score** and **Themes** (e.g., "Billing Issue", "Feature Request").

### **5. The Emergency Loop (Nexus)**
**File: [socketService.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/socket.js)**
- **What happens**: If the Sentiment is extremely negative, the `EventProcessor` bypasses the database and sends a live signal to the UI.
- **The Connection**: This is why the **Nexus Graph** on your dashboard glows red instantly.

---

### **Summary: The Chain**
`Ingest` $\rightarrow$ `PubSub` $\rightarrow$ `Processor` $\rightarrow$ `DLP/Gemini` $\rightarrow$ `Socket/DB`
