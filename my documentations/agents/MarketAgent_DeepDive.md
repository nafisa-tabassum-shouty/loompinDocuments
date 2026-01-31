# MarketAgent: The Competitive Intelligence Specialist

The `MarketAgent` is the **Competitive Intelligence Bureau** of the Loompin portal. Its job is to watch the "battlefield"—monitoring competitors, identifying feature gaps, and defining the "White Space" where Loompin can win.

---

## 1. What it does
The `MarketAgent` specializes in the `competitive_intelligence` domain. It is designed to solve the problem: *"How do we win against Competitor X?"*

**Key Responsibilities:**
*   **Competitor Battlecards:** Generating one-pagers on competitor strengths, weaknesses, and pricing.
*   **Feature Gap Analysis:** Identifying things competitors have that Loompin lacks (e.g., "Competitor X just launched automated reports").
*   **Win Strategy:** Developing specific sales pitches and strategic moves (e.g., "Highlight our AI speed vs their legacy bloat").
*   **Wargaming:** Simulating how competitors might react if Loompin drops its prices or launches a new feature.

---

## 2. Intelligence Logic: "The CSO's Brain"
The agent's personality and logic are defined by the **Chief Strategy Officer (CSO)** framework:

1.  **Mental Frameworks:** It uses "The 7 Powers" (Network Effects, Branding, etc.) and "Jobs-to-be-Done" (JTBD) to analyze products.
2.  **Emotional Subtext:** It doesn't just read reviews; it decodes the *emotion*. If it reads "This UI is too complex," it translates that into a **UX Vulnerability** for Loompin to exploit.
3.  **Ruthless Simulation:** It can act as a "Ruthless Competitor CEO" to help you prepare for price wars or smear campaigns.

---

## 3. Relationship with the "Market Analyst" Skill
The system currently uses two methods for this intelligence:

*   **`market-analyst.md` (Active):** This is a dynamic **Skill Agent**. It is actively loaded by the `AgentFactory` and the `Orchestrator`. It provides the high-level reasoning and "Strategic CSO" personality you see in the system.
*   **`MarketAgent.js` (Framework):** This is a structural **SmartAgent Template**. It provides the code loop (Recall -> Contextualize -> Hypothesize) but is currently not "switched on" in the main registration file. This allows for future hard-coded, high-performance logic while the system uses the more flexible `.md` skill currently.

---

## 4. Example Interaction: "The Counter-Attack"
1.  **User:** "We just launched a 20% discount. How will Competitor X react?"
2.  **PlanningAgent:** Recognizes the request as `competitive_intelligence` and routes it to the `market-analyst`.
3.  **Market Analyst:** Hypothesizes that Competitor X hates losing market share and has high "Process Power."
4.  **Investigation:** It searches recent pricing history and reviews for Competitor X.
5.  **Verdict:** "Competitor X will not match your price because of their high overhead. Instead, they will likely launch a 'Quality First' campaign. **Strategy:** Lean into our speed and lower cost as 'Modern Efficiency'."
