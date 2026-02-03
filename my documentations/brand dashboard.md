# Brand Admin Dashboard - Technical Overview

## 1. Data Source Analysis
The "Daily Briefing" and "NPS Score" sections currently displayed in the dashboard are **hardcoded** in the frontend component.

- **Component**: `frontend/src/components/dashboards/BrandAdminDashboard.tsx`
- **Location**: Lines 35-37 (Briefing) and 47-49 (NPS).

While the backend (`adminController.js`) provides real-time stats for total tenants, system health, and revenue, the specific narrative briefing and NPS trend are static placeholders for the current UI/UX demonstration.

## 2. NPS Score Explanation
- **Current Score**: `78`
- **What is NPS?**: Net Promoter Score (NPS) is a gold-standard customer loyalty metric. It is calculated by asking customers how likely they are to recommend a product on a scale of 0-10.
    - **Promoters (9-10)**: Loyal enthusiasts.
    - **Passives (7-8)**: Satisfied but unenthusiastic.
    - **Detractors (0-6)**: Unhappy customers.
- **Scoring**: NPS = (% Promoters) - (% Detractors). It ranges from -100 to +100.
- **Value of 78**: A score of 78 is considered **World Class**. Most top-tier SaaS companies aim for 50+, so 78 indicates extremely high brand advocacy.

## 3. Trend Indicator: "+4 vs last week"
- **Meaning**: This represents the "Delta" or improvement over the last 7 days.
- **Calculation**: It means the NPS score was `74` last week and has increased to `78` this week.
- **Significance**: A +4 increase in a single week is a significant positive trend, often linked to successful marketing campaigns (like the "Winter Sale" mentioned in the briefing) or improvements in customer support.

## 4. Implementation Note
To make this dynamic, the frontend should be updated to fetch these values from a new endpoint, such as `/api/analytics/nps` or by expanding the `getDashboardStats` response in the `adminController.js`.
