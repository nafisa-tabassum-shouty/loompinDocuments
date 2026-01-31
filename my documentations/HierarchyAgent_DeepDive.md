# HierarchyAgent: The Organizational Architect

The `HierarchyAgent` is a specialized intelligence agent designed to analyze and optimize organizational structures (people and departments). It acts as an **Organizational Design Consultant**, using high-reasoning LLMs to identify structural risks and inefficiencies.

> [!IMPORTANT]
> **This is NOT a file-system or project-structure agent.**
> It does not manage your code files, folders, or project architecture. It focuses strictly on the "People Hierarchy" of the business using the Loompin platform.

---

## 1. What it does
The `HierarchyAgent` (found in [HierarchyAgent.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/agents/squads/HierarchyAgent.js)) specializes in the `organization` domain. Its primary goals are:

*   **Structural Health Check:** Analyzing the "Span of Control" (how many people a manager manages).
*   **Anomaly Detection:** Identifying "orphan" units—departments that have high revenue or risk but are understaffed.
*   **Risk Clustering:** Finding groups of high-risk nodes (e.g., branches with high turnover or low customer satisfaction) within the organizational tree.
*   **Tree Reconcile:** Cleaning up and reorganizing hierarchies during data imports.

---

## 2. Intelligence Logic: "The McKinsey Brain"
The agent uses **Gemini 1.5 Pro** to perform deep organizational analysis. It doesn't just look at names; it looks at relationships:

1.  **Span of Control Anomalies:** It flags managers with more than 15 reports (micromanagement risk) or fewer than 3 reports (redundancy risk).
2.  **Health Scoring:** It generates a 0-100 `health_score` for an entire organization branch based on its balance.
3.  **Recommendations:** It provides high-level strategic advice, such as "Consolidate HR units in the NA Region" or "Audit the New York Branch for high risk density."

---

## 3. How it Works with Other Agents
While the `HierarchyAgent` is currently a standalone specialist, it is designed to be linked into the **Neural Core** orchestration:

### **Example Flow: Correlating Structure with Sentiment**
1.  **VoCAgent** detects a massive drop in customer sentiment in the "Retail South" region.
2.  **PlanningAgent** (the orchestrator) notices this and triggers the `HierarchyAgent`.
3.  **HierarchyAgent** analyzes the "Retail South" organizational tree.
4.  **Verdict:** It finds that the region has a manager with 25 reports—a "Span of Control" violation.
5.  **Synthesis:** The agents conclude that the sentiment drop is likely due to lack of manager oversight, not product issues.

### **Planned Integrations**
*   **[HierarchyService](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/services/HierarchyService.js):** The agent sits above this service. The service handles the SQL recursive queries to find "who reports to whom," while the agent provides the "Why" and "What should change."
*   **DataAnalystAgent:** Can provide the "Revenue/Risk" numbers for the `HierarchyAgent` to compare against headcount.

---

## 4. Key Cognitive Loop
| Stage | Action |
| :--- | :--- |
| **Contextualize** | Gathers the org tree and identifies the action (e.g., `analyze_structure`). |
| **Hypothesize** | Guesses where the friction might be (e.g., "span_of_control abnormality"). |
| **Investigate** | Flattens the tree and sends it to Gemini for expert analysis. |
| **Synthesize** | Produces a final `Health Report`, `Anomalies List`, and `Recommendations`. |
