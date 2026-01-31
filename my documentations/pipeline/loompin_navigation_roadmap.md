# 🗺️ Loompin VoC Codebase Learning Roadmap

The Loompin VoC platform is designed as a **"Neural Enterprise"**. To understand it quickly, don't read every file—instead, follow this functional flow:

---

## 🏗️ Phase 1: The "Senses" (How data gets IN)
*Start here to see how raw feedback becomes a "Unified Envelope".*
1.  **[UnifiedIngestionService.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/UnifiedIngestionService.js)**: The gatekeeper that standardizes all input.
2.  **[EventProcessor.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/EventProcessor.js)**: The pipeline that runs the SmartFilter, DLP (Redaction), and AI Enrichment.
3.  **[PubSubService.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loombot_VoC/backend/services/PubSubService.js)**: The async "nervous system" that moves data across the platform.

---

## 🧠 Phase 2: The "Thinking" (How the AI analyzes)
*Explore the cognitive architecture of the agent squads.*
1.  **[SmartAgent.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/agents/SmartAgent.js)**: The base "DNA" for all agents (Recall, Context, Hypothesis, Synthesis).
2.  **[AgentFactory.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/agents/squads/AgentFactory.js)**: The dynamic assembly line that creates agents from markdown skill files.
3.  **The Squads (`backend/agents/squads/`)**:
    - **ActionAgent**: Focuses on proactive responses.
    - **ChatAgent**: Focuses on conversational risk.
    - **DataAnalystAgent**: Focuses on SQL and visualization.

---

## 📊 Phase 3: The "Intelligence" (Deep Analysis Engines)
*See the specialized services that drive the platform's unique features.*
1.  **[NaturalQueryEngine.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/NaturalQueryEngine.js)**: The bridge between English and SQL.
2.  **[vectorStore.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/vectorStore.js)**: How the agents "remember" past interactions.
3.  **[ThematicService.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/neural-core/processing/ThematicService.js)**: The Gemini-powered theme and sentiment engine.

---

## 🎨 Phase 4: The "Body" (The Live Dashboard)
*How the frontend visualizes the "Nexus" loop.*
1.  **[NexusController.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/controllers/NexusController.js)**: How the backend serves the live topology graph.
2.  **`frontend/src/components/canvas/`**: The React-Three-Fiber tools for the interactive Strategy Deck and Nexus Graph.
3.  **[socket.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/socket.js)**: The real-time "heartbeat" that pushes signals to the UI.

---

## 💡 Pro-Tip for Navigation
Always look at the **`controllers/`** directory to see how a feature maps to an API route, and follow it back to the **`services/`** layer where the heavy lifting happens.
