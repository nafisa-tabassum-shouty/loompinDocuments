# Loompin VoC: Comprehensive Project Structure

This report provides a detailed mapping of the Loompin VoC (Voice of Customer) platform, categorized by functional layer and directory.

---

## 1. Root Directory & Configuration
The root contains the high-level orchestration, deployment, and configuration files.

| Directory / File | Description | Key Content |
| :--- | :--- | :--- |
| **`backend/`** | Node.js/Express Server | API, Agents, Services, Migrations. |
| **`frontend/`** | React/Vite/Tailwind UI | Features, Components, Global CSS. |
| **`terraform/`** | Infrastructure as Code | GCP/AWS resource definitions. |
| **`scripts/`** | Global DevOps Scripts | Population scripts, database seeds. |
| **`.agent/`** | Agentic Workflow Config | Skills and workflow definitions for Antigravity. |
| **`.github/`** | CI/CD Workflows | GitHub Actions for build/deploy. |
| **`docker-compose.yml`**| Service Orchestration | Local setup for DB, Redis, App. |
| **`package.json`** | Global Dependencies | Monorepo-style management. |

---

## 2. Backend: The Neural Core (`/backend`)
The backend is organized into functional layers for intelligence, logic, and data.

### 🧠 Intelligence & Agents
| Folder | Role | Key Files |
| :--- | :--- | :--- |
| **`agents/squads/`** | Specialized Agents | `ForecastingAgent.js`, `PlanningAgent.js`, `VoCAgent.js`. |
| **`skills/`** | Agent Personas | `market-analyst.md`, `security-guardian.md`, `data-scientist.md`. |
| **`services/`** | Logic Engines | `ForecastingEngine.js`, `NaturalQueryEngine.js`, `VectorStore.js`. |

### 📡 API & Controllers
| Folder | Role | Key Controllers |
| :--- | :--- | :--- |
| **`controllers/`** | Endpoint Logic | `intelligenceController.js`, `knowledgeController.js`, `analyticsController.js`. |
| **`routes/`** | API Mapping | `apiRoutes.js`, `platformRoutes.js`, `proxyRoutes.js`. |
| **`middleware/`** | Security & Auth | `authController.js`, `rbacController.js`, `twoFactorService.js`. |

### 🗄 Data Layer
| Folder | Role | Key Content |
| :--- | :--- | :--- |
| **`migrations/`** | Schema Evolution | `007_knowledge_graph.sql`, `011_digital_twin_schema.sql`. |
| **`db/`** | Connection Pooling | `db.js`. |
| **`schemas/`** | Validation Schemas | Zod or Joi schemas for request validation. |

---

## 3. Frontend: The Intelligence Panel (`/frontend`)
The frontend is feature-driven, where each module corresponds to a specific platform capability.

| Category | Feature Name | Description |
| :--- | :--- | :--- |
| **Strategic** | `StrategyDeck.tsx` | High-level business OKR and roadmap visualization. |
| **Analytics** | `FeedbackExplorer.tsx` | Deep-dive into customer sentiment and topics. |
| **Intelligence**| `KnowledgeBase.tsx` | Search and chat interface for the Knowledge Graph. |
| **Ops** | `SystemMonitor.tsx` | Health and telemetry for the agent squads. |
| **Customer** | `Customer360.tsx` | Unified view of a specific account's health and journey. |
| **Components** | `src/components/` | Reusable UI atoms (Buttons, Modals, Bento Grids). |

---

## 4. Specialized Directories
| Directory | Purpose |
| :--- | :--- |
| **`legacy/`** | Deprecated Code | Old modules kept for reference or backwards compatibility. |
| **`node_modules/`** | Dependencies | External libraries (handled by npm). |
| **`my documentations/`** | User Documentation | Human-readable deep-dives and architecture reports. |
| **`tests/`** | Quality Assurance | Integration and Unit test suites. |

---

## 5. Development Standards
*   **Design Manifest**: Strict adherence to semantic tokens (Tailwind) in `frontend/src/ui`.
*   **Cognitive Loop**: All `SmartAgents` follow the **6-stage loop** (Recall -> Contextualize -> Hypothesize -> Investigate -> Synthesize -> Self-Correction).
*   **Multi-Tenancy**: All database queries and API calls are scoped by `tenantId`.
