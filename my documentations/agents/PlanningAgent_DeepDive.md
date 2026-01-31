# PlanningAgent: The Lead Orchestrator

The `PlanningAgent` is the **Project Manager** of the Loompin Neural Core. While other agents specialize in specific domains (sentiment, data, strategy), the PlanningAgent specializes in **Orchestration**—breaking down complex human goals into simple machine tasks.

---

## 1. What it does
The `PlanningAgent` (found in [PlanningAgent.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/agents/squads/PlanningAgent.js)) is the primary entry point for any high-level request.

**Key Responsibilities:**
*   **Mission Decomposition:** Breaking a single prompt (e.g., "Why is our revenue dropping?") into 5-6 steps.
*   **Agent Selection:** Deciding which specialist (VoC, Analyst, Strategist) is best suited for each step.
*   **Sequential Execution:** Running tasks in order and passing the "baton" (data results) from one agent to the next.
*   **Result Synthesis:** Gathering all the "evidence" from specialists and writing the final report for the user.
*   **Design Guardian:** Ensuring that any UI/Frontend changes follow the strict design manifest.

---

## 2. The Orchestration Loop: "How it Plans"
When you send a request, the PlanningAgent goes through a 4-step process:

1.  **Decompose:** It asks Gemini to create a JSON `plan` array.
    - *Example Plan:* `[{ agentId: 'agent_voc_01', task: 'Analyze reviews' }, { agentId: 'agent_strat_01', task: 'Update roadmap' }]`
2.  **Execute:** It iterates through the plan. It calls `orchestrator.dispatch` for each step.
3.  **Context-Passing:** It takes the output from Step 1 and injects it into the prompt for Step 2. This allows agents to "talk" to each other indirectly.
4.  **Final Summary:** It looks at the execution history and writes a cohesive final answer.

---

## 3. Example Interaction: "The Revenue Rescue"
Here is how the PlanningAgent coordinates a squad of agents:

**Goal:** "My revenue in Europe is dropping and I don't know why. Fix it and update the roadmap."

| Step | Agent | Action | Result |
| :--- | :--- | :--- | :--- |
| **1** | **DataAnalyst** | Run SQL query on Europe revenue by product category. | Finds that "Product B" is down 40% in Germany. |
| **2** | **VoCAgent** | Analyze customer feedback specifically for "Product B" in Germany. | Finds high volume of complaints about "High Shipping Costs". |
| **3** | **MarketAgent** | Compare our shipping costs vs local competitors in Germany. | Finds Competitor X offers "Free Shipping" recently. |
| **4** | **StrategyAgent** | Calculate ROI of launching a 'Free Shipping' pilot in EU. | Determines it will restore 15% revenue growth. |
| **5** | **PlanningAgent** | **Synthesis:** "Your revenue dropped because of Competitor X's free shipping. I have updated the Roadmap to include a Free Shipping Pilot." | Final Report delivered to User. |

---

## 4. System Placement
| Integration | Role |
| :--- | :--- |
| **[AgentOrchestrator](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/agents/AgentOrchestrator.js)** | The Orchestrator "hands over" all high-level prompts to the PlanningAgent via `planAndExecute`. |
| **AgentFactory** | The PlanningAgent knows about every agent in the factory and their specific capabilities. |
| **Design Manifest** | The agent strictly enforces rounding, spacing, and semantic tokens for any UI-related tasks. |
