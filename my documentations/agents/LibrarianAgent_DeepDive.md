# LibrarianAgent: The Knowledge Custodian

The `LibrarianAgent` is the **Librarian of the Neural Core**. It is responsible for organizing, auditing, and retrieving the vast amount of unstructured knowledge (PDFs, docs, support tickets) flowing through the Loompin ecosystem.

---

## 1. What it does
The `LibrarianAgent` (found in [LibrarianAgent.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/agents/squads/LibrarianAgent.js)) operates in the `knowledge` squad. Its primary mission is to ensure that the "Collective Brain" of the platform is accurate and useful.

**Key Responsibilities:**
*   **Intelligent Ingestion:** Extracts a "Knowledge Graph" (entities like *Product: Loompin v2* and relations like *Policy: Refund Updated*) from uploaded documents.
*   **Knowledge Gap Detection:** Audits support tickets to find common questions that are *not* answered in the current library.
*   **Auto-Drafting:** Automatically drafts new Knowledge Base articles to fill those gaps.
*   **Stale Content Audit:** Reviews existing documents to find outdated policies or roadmap dates.

---

## 2. Intelligence Logic: "The Curator's Brain"
The agent uses **Gemini 1.5 Pro** to understand the structure of knowledge:

1.  **Entity-Relation Extraction:** It doesn't just index text; it understands the *context*. If it reads "Refunds are processed in 5 days," it creates nodes: `[Refund]` -- (hasPolicy) --> `[5 Day Processing]`.
2.  **Cluster Synthesis:** It works with the `VoCAgent` to understand themes. If the `VoCAgent` finds 50 tickets about "Login Issues," the Librarian checks the library, realizes there's no "Login Troubleshooting" guide, and initiates a `draft` task.
3.  **Chat Retrieval:** It acts as the back-end for the Knowledge Chat, finding relevant extracts from the `vectorStore` to answer user questions.

---

## 3. Interaction Example: The "Missing Feature" Loop
Here is how the Librarian interacts with other agents in a real-world scenario:

1.  **VoCAgent:** Clusters 200 customer signals and identifies a new "Theme": *Request for dark mode*.
2.  **LibrarianAgent:** Receives the cluster. It "Hypothesizes" that there's a knowledge gap.
3.  **LibrarianAgent:** It runs an "Audit" and "Investigation" phase where it searches the internal Roadmap and Knowledge Base.
4.  **Verdict:** It finds that "Dark Mode" is actually launching next week but is not yet documented.
5.  **Action:** It **Auto-Drafts** a new Help Article: *"How to enable Dark Mode (Coming Soon)"* and notifies the admin to review it.
6.  **StrategyAgent:** Sees the new article and includes it in its next Strategic Brief as a "Retention Win."

---

## 4. System Placement
| Integration | Role |
| :--- | :--- |
| **[knowledgeController](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/controllers/knowledgeController.js)** | Directly triggers the agent for every new document upload (`ingest`). |
| **[VectorStore](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/vectorStore.js)** | The Librarian feeds structured entities into the store for better semantic searching. |
| **AgentOrchestrator** | Routes "Knowledge" squad tasks specifically to this agent. |
