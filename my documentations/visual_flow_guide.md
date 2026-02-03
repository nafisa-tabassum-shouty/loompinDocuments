# Loompin: A Visual-First Technical Guide

To understand Loompin, it's best to start with what the user "sees" and then trace it down to the "Neural Core."

---

## 1. The Command Center (Frontend)
When you log in, you are greeted by the **Brand Admin Dashboard**. This is your "Mission Control."

- **Visual Component**: A Bento-style grid with active agents like "Strategy Prime" and "Sentiment Core."
- **How it works**: The frontend (`BrandAdminDashboardV2.tsx`) polls the `AgentOrchestrator` on the backend to show which AI agents are currently "thinking" or "active."
- **Why it matters**: It gives the user immediate visibility into the "Brain" of the platform.

---

## 2. The Data Import Flow (The "Doorway")
The **Data Import Wizard** is where raw customer data enters the world of Loompin.

### Step-by-Step Flow:
1. **Frontend**: You drag a CSV or JSON file into the [DataImportWizard](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/integrations/DataImportWizard.tsx).
2. **Standardization**: The frontend sends this to `/api/v1/ingest/upload/preview`. The backend's `UnifiedIngestionService` creates a "Unified Event Envelope."
3. **Commit**: When you click "Start Ingestion," the data is sent to the `EventProcessor`.
4. **AI Enrichment**: While the progress bar fills on your screen, the backend is running `AnalystAgent` and `SentimentAnalysis` to tag every row.

---

## 3. Real-Time Intelligence (The "Nerve System")
Loompin uses **WebSockets** to make the platform feel "alive."

- **The Visual**: The "Urgent Attention" card in the dashboard glows red when a negative signal (e.g., a tweet saying "Your app is down!") is detected.
- **The Backend Logic**:
  - The [EventProcessor.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/EventProcessor.js) detects a sentiment score below -50.
  - It immediately triggers `socketService.broadcast('nexus-signal', ...)`.
  - The frontend's `SystemMonitor` component receives this signal and updates the UI in milliseconds.

---

## 4. Summary Table

| What You See | Implementation Level | Key Backend Service |
| :--- | :--- | :--- |
| **3D Nexus Graph** | High-level Visualization | `ProductGraphService` |
| **Agent Fleet Status** | Orchestration Layer | `AgentOrchestrator` |
| **Import Progress Bar** | Data Ingestion Layer | `UnifiedIngestionService` |
| **Sentiment Gauges** | Intelligence Layer | `GeminiGateway` / `HuggingFaceEngine` |

---

## 5. How to Open the Brand Admin Dashboard
To see the dashboard in action, follow these steps:

1. **Start the Platform**: Open your terminal in the project root and run:
   ```bash
   npm run dev:all
   ```
2. **Access the UI**: Once the servers are up, open your browser to:
   [http://localhost:5173/login](http://localhost:5173/login)
3. **Quick Login**: On the login page, scroll down to the **"Development Override Protocols"** section and click the **"BRAND"** button.
4. **View Dashboard**: You will be immediately logged in and redirected to the [Brand Admin Dashboard](http://localhost:5173/dashboard).

---

## 6. How to Experience it Now (Files)
1. Open the [BrandAdminDashboardV2.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/dashboards/BrandAdminDashboardV2.tsx) to see the UI layout.
2. Trace a signal's journey by looking at [UnifiedIngestionService.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/UnifiedIngestionService.js).
3. See how agents collaborate in [AgentOrchestrator.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/agents/AgentOrchestrator.js).
