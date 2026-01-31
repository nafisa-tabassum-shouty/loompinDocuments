# Loompin Front-End Catalog

This catalog provides a comprehensive mapping of the Loompin VoC frontend architecture, organized by functional panels and dashboards.

## 1. Super Admin Panel
The Super Admin Panel is the control center for the entire platform, accessible only to root-level administrators. It handles multi-tenant operations, system health, and AI infrastructure.

| Page / Component | Directory Path |
| :--- | :--- |
| **Admin Overview** | [super-admin/AdminOverview.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/super-admin/AdminOverview.tsx) |
| **Tenant Management** | [super-admin/AdminTenants.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/super-admin/AdminTenants.tsx) |
| **AI Infrastructure** | [super-admin/AdminAI.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/super-admin/AdminAI.tsx) |
| **System Diagnostics** | [super-admin/AdminSystem.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/super-admin/AdminSystem.tsx) |
| **Platform Financials** | [super-admin/AdminFinancials.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/super-admin/AdminFinancials.tsx) |
| **Audit Log Viewer** | [super-admin/AdminLogs.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/super-admin/AdminLogs.tsx) |
| **Platform Team Control** | [super-admin/AdminTeam.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/super-admin/AdminTeam.tsx) |
| **Feature Switchboard** | [super-admin/AdminSwitchboard.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/super-admin/AdminSwitchboard.tsx) |
| **Feature Flag Manager** | [super-admin/FeatureFlagManager.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/super-admin/FeatureFlagManager.tsx) |
| **Plan Designer** | [super-admin/PlanDesigner.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/super-admin/PlanDesigner.tsx) |
| **AI Performance Monitoring** | [super-admin/AdminPerformance.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/super-admin/AdminPerformance.tsx) |

---

## 2. Main Dashboard & Role Views
These panels provide the landing experience for different user roles, varying from brand managers to local team leads.

| Page / Component | Directory Path |
| :--- | :--- |
| **Universal Dashboard** | [Dashboard.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/Dashboard.tsx) |
| **Brand Admin Dashboard** | [dashboards/BrandAdminDashboard.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/dashboards/BrandAdminDashboard.tsx) |
| **Manager Dashboard** | [dashboards/ManagerDashboard.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/dashboards/ManagerDashboard.tsx) |
| **Global Command (Super Admin)** | [dashboards/SuperAdminDashboard.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/dashboards/SuperAdminDashboard.tsx) |
| **Action Center (NBA)** | [ActionCenter.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/ActionCenter.tsx) |
| **Pulse (Real-time Feed)** | [Pulse.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/Pulse.tsx) |
| **Projects & Tasks** | [Projects.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/Projects.tsx) |

---

## 3. Analytics & Intelligence Panel
The Analytics layer provides deep dives into customer behavior, sentiment trends, and financial impact.

| Page / Component | Directory Path |
| :--- | :--- |
| **Customer 360 View** | [analytics/Customer360/Customer360View.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/analytics/Customer360/Customer360View.tsx) |
| **Intelligence Studio** | [analytics/IntelligenceStudio.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/analytics/IntelligenceStudio.tsx) |
| **Predictive Forecasting** | [analytics/PredictiveForecasting.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/analytics/PredictiveForecasting.tsx) |
| **Revenue Impact Analysis** | [analytics/RevenueImpact.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/analytics/RevenueImpact.tsx) |
| **ROI Simulator** | [analytics/ROISimulator.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/analytics/ROISimulator.tsx) |
| **Anomaly Radar** | [analytics/AnomalyRadar.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/analytics/AnomalyRadar.tsx) |
| **Deep Dive Analytics** | [analytics/DeepDiveAnalytics.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/analytics/DeepDiveAnalytics.tsx) |
| **Drivers Analysis** | [analytics/DriversAnalysis.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/analytics/DriversAnalysis.tsx) |

---

## 4. Survey & Inbound Engine
This panel manages the creation, distribution, and real-time rendering of adaptive customer surveys.

| Page / Component | Directory Path |
| :--- | :--- |
| **Survey Architect** | [SurveyBuilder.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/SurveyBuilder.tsx) |
| **Adaptive Survey Runner** | [survey/ChatSurveyRenderer.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/survey/ChatSurveyRenderer.tsx) |
| **Automated Distribution** | [SurveyDistribution.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/SurveyDistribution.tsx) |
| **Survey Logic Editor** | [SurveyLogicEditor.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/SurveyLogicEditor.tsx) |
| **Visual Flow Visualizer** | [SurveyFlowVisualizer.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/SurveyFlowVisualizer.tsx) |

---

## 5. Integration Hub
The Integration Hub handles the connection state and specific dashboards for various external data sources.

| Page / Component | Directory Path |
| :--- | :--- |
| **Main Integration Grid** | [Integrations.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/Integrations.tsx) |
| **CRM Specific Dashboard** | [integration-views/CRMDashboard.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/integration-views/CRMDashboard.tsx) |
| **Ecommerce Intelligence** | [integration-views/EcommerceDashboard.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/integration-views/EcommerceDashboard.tsx) |
| **Social Media Feed** | [integration-views/SocialDashboard.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/integration-views/SocialDashboard.tsx) |
| **Telephony Analytics** | [integration-views/TelephonyDashboard.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/integration-views/TelephonyDashboard.tsx) |

---

## 6. AI & Agent Interface
Specialized panels for interacting with the platform's Agentic Swarm.

| Page / Component | Directory Path |
| :--- | :--- |
| **Module Setup & Config** | [ModuleSetup.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/ModuleSetup.tsx) |
| **Persona Simulator** | [PersonaSimulator.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/PersonaSimulator.tsx) |
| **Simulation Studio** | [SimulationStudio.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/SimulationStudio.tsx) |
| **Pulse Assistant (AI)** | [ai/PulseAssistant.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/ai/PulseAssistant.tsx) |
| **Agent Draft Studio** | [agent-ui/AgentDraft.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/agent-ui/AgentDraft.tsx) |

---

## 7. Global Settings & Security
Tenant-wide configurations and identity management.

| Page / Component | Directory Path |
| :--- | :--- |
| **Tenant Settings** | [ClientSettings.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/ClientSettings.tsx) |
| **Brand Identity (UI/UX)** | [BrandSettings.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/BrandSettings.tsx) |
| **Compliance & Privacy** | [ComplianceCenter.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/ComplianceCenter.tsx) |
| **Role & Security Config** | [RoleConfig.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/RoleConfig.tsx) |
| **Org Units & Hierarchy** | [Hierarchy.tsx](file:///Users/user/Desktop/PC/anti%20gravity%20projects/loompin/loompin_VoC/frontend/src/components/Hierarchy.tsx) |
