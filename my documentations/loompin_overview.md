# Loompin VoC Platform: Technical Overview

Loompin is a next-generation **Voice of Customer (VoC)** and **Market Analysis** platform designed to centralize customer signals from disparate channels into a single "Intelligence Hub."

## 1. Core Architecture: The "Neural Core"

The system follows a **Monorepo Architecture** with a Node.js/Express backend and a React/Vite frontend.

- **Backend (`/backend`)**: Orchestrates the "Neural Core," which handles data ingestion, AI processing, and agentic orchestration.
- **Frontend (`/frontend`)**: The "Intelligence Panel," providing real-time visualizations like the **Nexus Graph** (3D data representation).
- **Agent Swarm**: A collection of specialized AI agents (e.g., `AnalystAgent`, `EmailAgent`, `VideoAgent`) that collaborate via an `AgentOrchestrator`.

---

## 2. Featured Implementation: Unified Ingestion & AI Enrichment

The most critical flow in the system is how it processes raw customer signals.

### Backend: The "Customs Officer" Pipeline
The `UnifiedIngestionService` and `EventProcessor` work together to transform raw data into "Intelligence":

1. **Ingestion**: Data enters via the `UnifiedIngestionService`. It standardizes the input into a "Unified Event Envelope" (including `trace_id`, `tenant_id`, and `payload`).
2. **Filtering**: The `smartFilter` evaluates if the signal is noise, high-impact, or needs aggregation.
3. **AI Enrichment**:
   - If it's a **Call Recording**, the `audioService` generates a transcript, summary, and sentiment.
   - If it's an **Email**, the `EmailAgent` analyzes the thread for intent and sentiment.
   - If it's **Video**, the `VideoAgent` extracts visual tags and summaries.
4. **Safety & Compliance**: The `dlpService` automatically redacts PII (Personally Identifiable Information) before storage.
5. **Storage**: The `DataFactory` routes the processed signal to the appropriate data store (PostgreSQL for hot data, BigQuery for cold analytics).

### Frontend: The "Data Import Wizard"
The user interacts with this through the `DataImportWizard.tsx` and views the results in the `OmniChannelView.tsx`.

- **Real-time Feedback**: The system uses WebSockets (`socketService`) to broadcast "nexus-signals" back to the frontend.
- **Visual Evidence**: Critical signals (e.g., extremely negative sentiment) are immediately highlighted in the dashbard.

---

## 3. Current Feature Status

| Feature Pillar | Key Components | Completion Status |
| :--- | :--- | :--- |
| **Ingestion** | Multi-channel scripts, Unified Ingestion API | ✅ Complete |
| **AI Processing** | Agent Swarm, Hugging Face/Gemini failover | ✅ Complete |
| **Visualization** | Nexus 3D Graph, Omni-channel View | ✅ Complete |
| **Market Analysis** | Reddit Scraper, Competitor Monitor | ✅ Complete |
| **Infrastructure** | GCP Hyper-Scale, Pub/Sub Pipeline | ✅ Complete |
| **Advanced UI** | 3D Visualization (3D-Nexus) | 🚧 In Progress |

---

## 4. How to Run the Project

The project is designed for seamless local development.

### Option A: Local Execution (Recommended for Dev)
1. **Initialize Services**:
   ```bash
   npm run setup
   ```
   *This script handles Docker/Postman runtime detection, provisions PostgreSQL and Redis, and runs migrations.*

2. **Start Dev Environment**:
   ```bash
   npm run dev:all
   ```
   *This starts both the backend (Express) and frontend (Vite) simultaneously.*

### Option B: Docker Compose
If you prefer a containerized environment:
```bash
docker-compose up --build
```

---

## 5. Project Roadmap & Planning

The project has transitioned from a flat-file prototype to a **GCP Hyper-Scale Architecture**. Future planning includes:
- **Nexus Graph refinement**: Enhancing the 3D visualization for large-scale data sets.
- **BYOD (Bring Your Own Database)**: Implementing data sovereignty patterns for enterprise clients.
- **Predictive Churn Engine**: Deepening the "Risk Genome" analysis for customer profiles.
