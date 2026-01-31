# PredictionAgent: The Risk Guardian

The `PredictionAgent` is the **Probability Specialist** of the Loompin ecosystem. Its primary mission is to look at customer history and calculate the statistical likelihood of future events—most notably **Churn Risk**.

---

## 1. What it does
The `PredictionAgent` (found in [PredictionAgent.js](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/backend/agents/squads/PredictionAgent.js)) operates in the `predictive_analytics` domain. It is an "Advance Guard" agent designed to protect the business from surprises.

**Key Responsibilities:**
*   **Churn Probability:** Calculating a 0-100 score on how likely a customer is to cancel.
*   **Risk Factor Diagnosis:** Identifying *exactly why* a customer is at risk (e.g., "3 Billing tickets this week").
*   **Next Best Action (NBA):** Deciding if a risk requires a simple automated email or a "Ruthless Executive Outreach."
*   **Upsell Identification:** Finding customers who are happy and healthy enough for a higher tier.

---

## 2. Intelligence Logic: "The Neural Predictor"
The agent uses a data-driven cognitive loop to reach its verdicts:

1.  **Hypothesize:** It looks for specific "Risk Patterns" in context, such as declining sentiment combined with a spike in support tickets.
2.  **Investigate:** It runs an inference (Simulation or BigQuery ML) to calculate a `riskScore`.
3.  **Cross-Referencing:** It checks the `history` for "Hostile" recordings or sharp NPS drops to back up its score with evidence.
4.  **Synthesize:** It produces a **Prediction Report** that includes not just the score, but the "Why" and the "What to do now."

---

## 3. How it Works with Other Agents
The `PredictionAgent` acts as a data-feeder for more senior agents:

### **Example Flow: The Rescue Mission**
1.  **AnomalyAgent:** Detects a sudden 30% drop in user activity for an Enterprise account.
2.  **PlanningAgent:** Receives the alert and triggers the `PredictionAgent`.
3.  **PredictionAgent:** Analyzes the account's last 90 days.
4.  **Verdict:** "90% Churn Probability. **Reason:** Unresolved technical debt in their private instance. **NBA:** Executive Outreach."
5.  **StrategyAgent:** Receives this verdict and automatically adjusts the Quarter Roadmap to prioritize the "Enterprise Stability" bug fix.

---

## 4. Implementation Status
> [!NOTE]
> **Advanced Tier Agent:**  
> The `PredictionAgent` is currently a **Specialized Framework**. It exists in the code but is often triggered by specific high-level services like the `Intelligence Dashboard` rather than being part of the standard daily "Chat" squad. This keeps the daily chat fast while reserving the "Neural Predictor" for high-stakes account reviews.
