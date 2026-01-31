# Controller Catalog

This document provides a comprehensive catalog and detailed breakdown of the controllers within the Loompin VoC backend, explaining their functionalities, logic, and implementation details.

---

## AdminController (`adminController.js`)

The `AdminController` is a central hub for platform-level administrative operations. It manages system-wide statistics, health monitoring, telemetry, global settings, feature flagging, tenant provisioning, and high-privilege operations like user impersonation ("God Mode").

### How it Works
The controller leverages asynchronous patterns (`async/await`) and parallel processing (`Promise.all`) to efficiently query the database for various system metrics. It interacts with several services including the database helper (`db.js`), a centralized `logger`, `AuditLogService`, and `QueueService`. It serves as the backend logic for the Super Admin Dashboard.

### Functionalities and Features

1.  **Dashboard Stats Aggregation**: Collects total tenant counts, recent growth, revenue estimates (ARR), and global AI token consumption.
2.  **System Health Monitoring**: Measures database latency, process uptime, and memory usage to ensure platform stability.
3.  **Telemetry & Safety Tracking**: Monitors system events and tracks safety violations or blocked requests.
4.  **Global System Settings**: Manages platform-wide configurations like maintenance mode, AI provider priorities, and global banners.
5.  **Feature Flag Management**: Provides full CRUD operations for enabling/disabling features dynamically across the platform.
6.  **User Impersonation (God Mode)**: Allows super admins to generate valid JWTs for any user or create "phantom" sessions for troubleshooting.
7.  **Subscription Plan Management**: Manages available service plans and pricing tiers.
8.  **Tenant Provisioning**: Orchestrates the creation of new tenants, including infrastructure metadata (shards, buckets) and initial admin users.
9.  **Audit Log Access**: Provides visibility into administrative actions and system events.
10. **Queue Monitoring**: Tracks the status of background jobs (e.g., email, SMS) across different queues.
11. **Platform Team Management**: Retrieves details of the internal team managing the platform.
12. **AI Insights**: Exposes AI-generated anomalies or recommendations for system optimization.

### Feature Logic Descriptions

-   **Dashboard Stats**: Uses parallel SQL queries via `Promise.all` to fetch tenant counts, plan distributions, and token usage simultaneously. Revenue is calculated by multiplying plan counts by predefined price points to estimate Annual Recurring Revenue (ARR).
-   **System Health**: Dynamically measures the time taken for a simple "SELECT 1" query to determine DB latency. It pulls process-level diagnostics using Node.js `process.uptime()` and `process.memoryUsage()`.
-   **Impersonation**: Generates a new JWT containing a special `impersonator_id` claim. It supports "Phantom" sessions which are transient user objects not necessarily stored in the main user table but valid for the session duration.
-   **Tenant Provisioning**: Generates a unique tenant ID (`tnt-uuid`), assigns infrastructure metadata (random DB shards and unique S3 buckets), and creates a linked administrative user in the same transaction flow.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 1-4 | **Foundation Initialization**: Imports `jsonwebtoken` for secure token management, `query` for database interactions, `logger` for observability, and `uuid` for generating unique identifiers. These dependencies are essential for creating the secure, traceable, and unique records required for administrative actions. |
| 6-16 | **Parallel Data Aggregation**: The `getDashboardStats` function uses `Promise.all` to execute multiple independent queries (Tenant counts, Plan distributions, Global token usage) simultaneously. This architectural choice significantly reduces latency by avoiding sequential database round-trips, ensuring the Admin Dashboard loads quickly even as the scale of data increases. |
| 31-57 | **Business Metric Computation**: Instead of fetching static values, the logic calculates ARR (Annual Recurring Revenue) on-the-fly by mapping tenant counts to pricing tiers. This ensures that the financial overview is always perfectly synced with the current subscription state, while formatting functions (like dividing by 1000 for 'k') make the data human-readable for high-level decision makers. |
| 66-75 | **Connectivity and Resource Probing**: Measures system health by timing a `SELECT 1` query to determine database latency and fetching process-level metrics using Node.js `process.uptime()` and `process.memoryUsage()`. This provides a dual-layer health check: ensuring the database is responsive while monitoring the physical resource health of the application server itself. |
| 112-134 | **Telemetry Analysis Flow**: Aggregates recent system events with specific focus on average request duration and safety violations. By calculating the average latency from JSONB metadata (`metadata->>'duration'`), the controller provides real-time performance monitoring alongside security observability, allowing admins to correlate spikes in errors with specific system events. |
| 141-182 | **Stateful Configuration Management**: Implements a "Fetch or Default" pattern for system settings. If the database is empty, it returns a hardcoded safe state. When updating, it uses logger to record the email of the admin making the change, creating a non-repudiation layer for sensitive platform configuration changes like Maintenance Mode. |
| 244-300 | **Secure Impersonation (God Mode)**: Implements troubleshooting logic that generates a special JWT containing an `impersonator_id`. This allows an admin to act on behalf of a user (or as a role-based 'Phantom') while maintaining a cryptographic link to the original admin's identity for auditing. It supports both ID-based and tenant-first lookup for maximum flexibility during support calls. |
| 330-400 | **Orchestrated Provisioning Pipeline**: This logic standardizes the birth of a new tenant. It generates a unique `tnt-` ID, assigns infrastructure shards (DB/S3), and creates the initial admin user in a single cohesive flow. This ensures that infrastructure metadata and user identity are atomically linked, preventing "orphaned" tenants or users without resource assignments. |
| 406-446 | **Operational Observability Service**: These endpoints (Audit, Queues, Team, Insights) serve as lightweight proxies to specialized services. By delegating to `AuditLogService` and `QueueService`, the controller keeps the API layer thin while ensuring that complex logic like job counting or staff lookup is handled by the appropriate domain-driven module. |

---

## AgentController (`agentController.js`)

The `AgentController` manages the lifecycle and tasking of autonomous AI agents and squads. It acts as the bridge between the API layer and the `AgentOrchestrator`, while also handling agent memory mechanisms like feedback loops and consolidation.

### How it Works
The controller interacts directly with the `AgentOrchestrator` to dispatch jobs to specific "Squads" (e.g., Strategy, VoC). It also uses the `AgentFactory` to reflect on available agent capabilities. For memory features, it interacts with the `agent_episodes` database table to store and analyze agent performance.

### Functionalities and Features

1.  **Real-time Agent Status**: Polls the orchestrator for the total number of live agents, active squads, and detailed skill availability.
2.  **Strategic Task Dispatch**: Specifically routes strategic and market analysis requests to the "Strategy" squad.
3.  **VoC Task Dispatch**: Routes survey design and feedback analysis requests to the "VoC" squad.
4.  **Reinforcement Learning Feedback**: Provides an endpoint for human-in-the-loop feedback where users can rate agent performance (-1 to 1).
5.  **Memory Consolidation**: A background-ready task that clusters recent agent experiences to find recurring patterns or "lessons learned."

### Feature Logic Descriptions

-   **Task Dispatching**: The `orchestrator.dispatch` method is central. It takes a JSON payload describing the goal and routes it to the first available agent in the specified squad that can handle that "type" of request.
-   **Skill Discovery**: Uses `AgentFactory.getAllAgents()` to dynamically list what agents are currently registered in the system (e.g., Analyst, Product Manager).
-   **Memory Feedback Loop**: Updates the `outcome_rating` in `agent_episodes`. This rating is crucial for future agent self-optimization (RLHF equivalent for the agentic layer).
-   **Clustering Logic**: (MVP Phase) Queries episodes from the last 24 hours. The logic is designed to use embeddings (via a vector service) to find dense clusters of similar tasks to generate new automated workflows or rules.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 2-4 | **Agent Hub Dependencies**: Imports the database query helper for persistence, the `orchestrator` for high-level task routing, and the `AgentFactory` for skill metadata. This setup decouples the API layer from the complex agentic logic, allowing the controller to act as a clean mediator. |
| 6-23 | **Ecosystem Status Mapping**: The `getAgentStatus` function reflects on the current state of the "Brain." It queries the `AgentFactory` to see which specialized skill agents (Analyst, Researcher, etc.) are available and then maps those into a detailed status object. This gives frontend users visibility into which cognitive capabilities are currently online or active. |
| 31-49 | **Squad-Based Tasking**: These endpoints (`runStrategyAgent`, `runMarketAgent`, `runSurveyAgent`) serve as specialized entry points into the agent swarm. By dispatching tasks to specific squads (Strategy vs. VoC), the controller ensures that requests are handled by agents with the relevant specialized training and context, improving accuracy and efficiency. |
| 62-75 | **Experience-Based Memory Loop**: Implements the human-in-the-loop feedback mechanism. Users submit a rating (-1, 0, or 1) which is persisted to the `agent_episodes` table. This logic is critical for autonomous self-correction; the system uses these ratings to understand which agent behaviors were successful, forming a long-term supervised learning signal for the entire swarm. |
| 92-103 | **Memory Consolidation Process**: Triggers a job that analyzes agent episodes from the last 24 hours. The goal of this logic is to find dense clusters in high-dimensional vector space where agents succeeded or failed, essentially "meditating" on recent experiences to extract higher-level rules or automated workflows. |

---

## AIController (`aiController.js`)

The `AIController` is the primary interface for generic Low-Level Intelligence (LLI) tasks. It abstracts away the complexities of interacting with Large Language Models (LLMs) and provides a unified API for analysis and summarization.

### How it Works
It delegates all LLM execution to the `GeminiGateway`, which handles model selection, rate limiting, and prompt wrapping. The controller focuses on request validation, complexity routing, and response sanitization (particularly for JSON-based outputs).

### Functionalities and Features

1.  **Universal Analysis**: A highly flexible endpoint that handles prompts for sentiment analysis, theme extraction, data tagging, and more.
2.  **Summarization Engine**: A dedicated pathway for compressing large volumes of text into concise summaries.
3.  **Complexity Routing**: Automatically routes requests to "Flash" (Low Complexity) or "Pro" (High Complexity) models based on user requirements.
4.  **Automatic JSON Sanitization**: Detects and cleans LLM outputs that contain markdown code blocks (e.g., ` ```json `) before returning raw JSON to the client.

### Feature Logic Descriptions

-   **Complexity Determination**: Defaults to 'low' for maximum speed and cost efficiency. If a user needs higher reasoning for a "Deep Analysis," they can specify `complexity: 'high'` in the config.
-   **JSON Mode Enforcement**: When `responseMimeType` is set to JSON, it instructs the gateway to use specific model parameters for structured output and then double-checks the result for parsing errors.
-   **Error Shielding**: Uses a 503 (Service Unavailable) status if the AI provider is down, distinguishing it from internal application errors.

### Line-by-Line Explanation

| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 4 | **Core AI Gateway Access**: Imports the `geminiGateway`, which abstracts all direct LLM interactions. This ensures that the controller doesn't need to handle API keys, retry logic, or provider-specific formatting, focusing solely on the high-level analysis intent. |
| 8-14 | **Intelligent Routing Logic**: The `analyze` endpoint extracts the prompt and determines the required reasoning power. By defaulting to 'low' complexity but allowing 'high' overrides, the controller implements a cost-optimization strategy: using fast, cheap models for simple tasks and reserving powerful models for complex thematic deep-dives. |
| 16-21 | **Generation Dispatch**: Invokes the gateway's `generate` method, passing the system instructions and complexity settings. This separation of concerns allows the AI platform to evolve (e.g., swapping models) without changing the controller's implementation. |
| 26-31 | **JSON Determinism Sanitization**: Large Language Models often wrap valid JSON in markdown fences (e.g., ` ```json `). This logic detects such patterns, strips the fences, and parses the result. If parsing fails, it logs a non-blocking warning, ensuring the system can fall back to returning raw text instead of crashing the request. |
| 36 | **Final Delivery**: Delivers the sanitized and structured analysis to the client. The use of a `finalResult` variable allows for easy post-processing or metadata injection (like token counts) before the final response is sent. |
| 44-50 | **Optimized Summarization Flow**: A specialized path for high-volume text compression. It forces `complexity: 'low'` because summarization is a pattern-extraction task where the speed and context window of a 'Flash' class model outperform the heavy reasoning of a 'Pro' class model. |

---

## AnalyticsController (`analyticsController.js`)

The `AnalyticsController` is the core engine for generating data-driven insights, visualizations, and strategic recommendations. It orchestrates complex data aggregations and interacts with the AI agent swarm for advanced analysis.

### How it Works
It leverages the `orchestrator` to delegate tasks to specialized agents (like `voc` or `analytics`) and uses service-layer components like `scenarioService` and `recommendationEngine`. It handles both real-time AI queries ("Ask AI") and pre-aggregated dashboard data.

### Functionalities and Features

1.  **getConversationClusters**: Triggers the VoC (Voice of Customer) Agent to perform unsupervised clustering on conversation signals to identify emergent themes.
2.  **getDriverTree**: Retrieves the "Experience Driver Tree," mapping topics to their impact scores, contribution weights, sentiment, and volume.
3.  **getDriverHistory**: Provides time-series history for a specific driver, correlating its impact with core metrics over time.
4.  **getJourneyFlow**: Exports node-link data representing the customer journey path through various platform touchpoints.
5.  **getForecast**: Endpoint reserved for predictive forecasting metrics (Phase 6 implementation).
6.  **getAnomalies**: Endpoint reserved for reporting strategic anomalies detected by the AnomalyAgent.
7.  **getMissions**: Aggregates actionable "Missions" derived from deep driver analysis.
8.  **launchMission**: Records a Super Admin's decision to execute a specific strategic mission in the audit logs.
9.  **getNextBestAction (NBA)**: Leverages the "Cortex" (Unified processing core) to deliberate and suggest the most impactful next step for a tenant.
10. **submitFeedback**: Captures manual feedback on agent-suggested actions to refine the reinforcement learning loop.
11. **getMetrics**: A high-level dashboard aggregator for NPS (Surveys), CSAT (Events), and Churn Risk (Strategy maps).
12. **askAI**: A natural language data assistant that dispatches questions to the 'analytics' squad, returning data, SQL, and chart configurations.
13. **getDashboardChart**: Provides a 7-day trailing count of customer events for volume trend visualization.
14. **getThemes**: Retrieves categorized thematic insights (Placeholder for future thematic engine).
15. **getRadarData**: Fetches multi-dimensional segment perception data for visualization in radar/spider charts.
16. **getFunnelData**: Automatically maps behavioral events into a conversion funnel (Awareness -> Interest -> Decision -> Action).
17. **getHeatmapData**: Generates a 7x24 matrix of user activity density (Day of Week vs. Hour of Day).
18. **runSimulation**: Interfaces with the `scenarioService` to run mathematical simulations of metric changes based on driver adjustments.
19. **getRecommendations**: Serves optimized strategic recommendations through the `recommendationEngine`.
20. **getRecentFeedback**: Provides a live stream of the most recent agent reflex actions and their outcomes.
21. **getConversations**: Lists detailed conversation logs including AI-generated sentiment journeys, themes, and categories.
22. **getCrosstabData**: Performs a pivot analysis between event types and sentiment segments (Promoters/Passives/Detractors).

### Feature Logic Descriptions

-   **Unsupervised Clustering**: The `getConversationClusters` dispatch sends an empty data array to the `voc` agent, which is expected to independently pull raw signals, perform clustering via embeddings, and return structured theme objects.
-   **Strategic Deliberation**: `getNextBestAction` does not just retrieve data; it creates a `strategic_review` signal that passed through the `cortexOrchestrator`, mimicking a human brainstorming session to arrive at a recommendation.
-   **Funnel Mapping**: The funnel logic is dynamic. It searches the `customer_events` table for specific lifecycle milestones and maps them to a fixed 4-stage marketing funnel, calculating conversion counts for each.
-   **Heatmap Matrix Construction**: The controller manually builds a "dense" matrix. It queries the database for active cells (Day/Hour pairs) and then fills in the gaps (0s) for all 168 hours of the week in JavaScript to ensure the frontend grid is complete.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 1-5 | **High-Dimensional Intelligence Imports**: Imports essential services for simulation, recommendations, and the Cortex orchestrator. These specialized services empower the controller to move beyond simple data retrieval and into the realm of outcome forecasting and strategic deliberation. |
| 10-22 | **Autonomous Theme Discovery**: Dispatches an unsupervised clustering task to the VoC squad. The logic assumes that agents will analyze raw conversation signals to group them into thematic clusters. The controller includes a safeguard that returns an empty array instead of failing, ensuring a resilient user experience if the agent swarm is temporarily overloaded. |
| 44-88 | **Driver Impact Analysis**: Implements a correlation engine between topics and business impact. By joining `experience_drivers` with `driver_history`, the logic provides both a snapshot of current sentiment and a time-series view of how a specific driver's weight has evolved, allowing admins to track the long-term effectiveness of their strategic interventions. |
| 155-163 | **Traceable Strategic Action**: When a Super Admin launches a "Mission," the controller records it in the `audit_logs`. This logic ensures that every high-level strategic decision is cryptographically tied to an administrator's identity, providing a clear chain of accountability for platform-level changes. |
| 171-176 | **Cortex-Driven Strategic Review**: Unlike standard queries, `getNextBestAction` initiates a `strategic_review` through the Unified Brain (Cortex). This simulates a deliberative process where multiple signals (churn, sentiment, NPS) are cross-referenced to arrive at the single most impactful recommendation for the tenant. |
| 218-248 | **Main Dashboard Synthesis**: Aggregates disparate health signals (NPS from surveys, CSAT from events) into a unified metrics view. This calculation logic standardizes different data sources into a consistent set of labels, providing a "Single Source of Truth" for the brand's performance overview. |
| 263-280 | **NLQ-to-Insight Transformation**: The `askAI` logic acts as a bridge between human language and structured data. It delegates the complexity of SQL generation and chart-type determination to the 'analytics' squad, then returns a "Smart Result" that includes the raw data along with the AI's explanation of its findings. |
| 359-391 | **Behavioral Funnel Mapping**: Dynamically tracks user progression through fixed lifecycle stages (Awareness to Action). The SQL query identifies unique reach at each stage by filtering `customer_events` for specific milestones, and the JavaScript logic then re-orders these results into a logical funnel sequence for visualization. |
| 409-438 | **Activity Density Calculation**: Generates a 7x24 heatmap of platform engagement. The logic extracts the Day-of-Week and Hour from events and then uses a nested loop to "backfill" missing hours with zeros. This ensures the resulting matrix is always complete, preventing broken or uneven grids in the frontend visualization. |
| 509-531 | **Customer Sentiment Journey**: Fetches detailed conversation logs enriched with AI metadata. The logic carefully extracts the `sentiment_journey` and `summary` objects, allowing the frontend to render a cinematic view of individual customer interactions and thematic progression. |
| 565-620 | **Singleton Instance & Method Binding**: Exports a singleton instance of the controller and manually binds its methods. This architectural pattern ensures that `this` context is preserved when methods are passed directly to router functions, while also optimizing memory by reusing the same controller instance across all requests. |

---

---

## AuditController (`auditController.js`)

The `AuditController` provides high-level transparency into administrative actions by exposing the platform's security vault.

### How it Works
It is a read-only controller that interfaces directly with the `AuditLogService`. It is typically mapped to a Super Admin route to allow platform overseers to review sensitive configuration changes and mission launches.

### Functionalities and Features
1.  **getAuditLogs**: Retrieves the entire history of system-audit events, providing a chronological trail of "who did what and when."

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 2-12 | **Security Vault Retrieval**: Interfaces with the `AuditLogService` to provide platform overseers with a chronological ledger of sensitive actions. The logic implements a descending sort (`b.id - a.id`) to ensure that the most recent (and potentially most urgent) security events are displayed first, optimizing the time-to-detection for audit reviews and system monitoring. |

---

## AuthController (`authController.js`)

The `AuthController` manages the primary security barrier of Loompin VoC, handling user identity, secure credential storage, and session issuance.

### How it Works
It employs a multi-layered security approach, using `crypto` for `scrypt` hashing on new registrations while maintaining `bcrypt` compatibility for legacy accounts. It issues stateful JWTs that carry tenant and role metadata, enabling RBAC (Role-Based Access Control) across the entire system.

### Functionalities and Features
1.  **hashPassword (Internal)**: Uses the `scrypt` algorithm with a 16-byte random salt to generate computationally expensive password hashes.
2.  **verifyPassword (Internal)**: A robust verification helper that detects the hash format (`bcrypt` vs. `scrypt`) and uses `timingSafeEqual` to prevent timing attacks.
3.  **register**: A complex workflow that checks for user existence, provisions a new tenant organization with default branding, creates a "Brand Admin" user, and issues an immediate session token.
4.  **login**: Validates credentials, creates a tenant-contextual payload, and signs a 24-hour JWT for the user.
5.  **getMe**: A "Who Am I" endpoint that returns the authenticated user's profile and current tenant association.

### Feature Logic Descriptions
-   **Atomic Provisioning**: During `register`, the controller performs sequential database inserts for both the `tenants` and `users` tables. If the user creation fails, it ensures the platform has a record of the associated tenant for administrative cleanup or manual completion.
-   **Security Hardening**: The `verifyPassword` function uses `timingSafeEqual`. This ensures that the time taken to compare a password doesn't reveal how many characters were correct, effectively neutralizing a common side-channel attack vector.
-   **Phantom Detection**: While `AuthController` handles standard logins, the JWT format it establishes is designed to be compatible with the "Phantom Sessions" issued by the `AdminController` during impersonation.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 1-6 | **Security Stack Foundations**: Initializes the auth layer with industry-standard libraries for token issuance (JWT) and cryptographic security (crypto, bcrypt). By wrapping endpoints in `asyncHandler`, the controller ensures that any authentication failure or database timeout is gracefully handled and logged without crashing the process. |
| 11-37 | **Hybrid Cryptographic Verification**: Implements a robust "Legacy-to-Modern" password strategy. The logic detects the hash format (`bcrypt` vs. `scrypt`) and routes to the appropriate verification function. Using `timingSafeEqual` for scrypt hashes is a critical security measure that neutralizes timing side-channel attacks by ensuring comparisons always take the same amount of time regardless of how many characters are correct. |
| 40-88 | **Atomic Onboarding Workflow**: Handles the sequential creation of a tenant and its primary administrator. The logic ensures that a `tnt-` workspace is initialized with Enterprise defaults and that the user is assigned the 'Brand Admin' role immediately. If the user creation fails after the tenant insert, the transaction ensures the system is left in a state where the registration can be resumed or audited. |
| 93-124 | **Sanitized Session Issuance**: Validates user credentials and issues a stateful 24-hour JWT. A key security step included in this logic is the explicit `delete user.password` call; this ensures that hashed credentials never leave the server's memory, even as part of a successful login response, minimizing the risk of credential leakage. |
| 129-134 | **Identity Synchronization**: The `getMe` logic retrieves the full profile of the authenticated user based on the ID extracted from the high-trust JWT. This ensures that the frontend always has a fresh, server-verified view of the user's role and tenant membership, preventing local session manipulation. |

---

---

## BillingController (`billingController.js`)

The `BillingController` handles the commercial lifecycle of the platform, automating tenant onboarding and status changes based on payment events.

### How it Works
It utilizes the Stripe Webhook architecture to listen for external events. It maps these events to internal state transitions, ensuring that successful payments trigger immediate provisioning and cancellations lead to secure resource suspension.

### Functionalities and Features
1.  **handleStripeWebhook**: Orchestrates the routing of incoming Stripe events to specialized handlers based on event types.
2.  **handleCheckoutCompleted (Internal)**: Processes successful subscription completions. It determines if a payment is for a new or existing user and triggers the appropriate upgrade or provisioning logic.
3.  **handleSubscriptionDeleted (Internal)**: Automatically sets a tenant's status to 'Suspended' when a subscription is canceled at the provider level.

### Feature Logic Descriptions
-   **Smart Provisioning Logic**: In `handleCheckoutCompleted`, the system first checks the `users` table. If the email is recognized, it upgrades the existing tenant (`UPGRADE`). If not, it generates a fresh `tnt-` workspace, a `pk_live` API key (hashed for security), and an admin account (`PROVISION`).
-   **API Key Hardening**: When a new tenant is provisioned, the controller generates a 24-byte cryptographically secure secret, hashes it using SHA-256, and stores only the hash. This follows security best-practices where even a DB breach does not expose active API secrets.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 2-20 | **Webhook Orchestration Strategy**: Imports database and security utilities to handle external payment signals. The `handleStripeWebhook` function implements a switch-case router that dispatches incoming events to specialized internal handlers. This architecture ensures that the webhook endpoint remains responsive while offloading complex state changes to dedicated sub-functions. |
| 32-51 | **Context-Aware Upgrade Logic**: When a checkout is completed, the logic first probes for an existing user. If found, it executes an `UPDATE` on the `tenants` table to immediately unlock 'Growth' plan features. This "Instant Upgrade" flow improves user satisfaction by removing manual intervention from the subscription lifecycle. |
| 55-82 | **Cryptographic Provisioning Pipeline**: For new users, the controller orchestrates a complex insert sequence. A key security feature is the generation of a 24-byte `randomBytes` secret, which is then SHA-256 hashed before storage. This ensures that the tenant's primary API key is initialized with high entropy, meeting modern security standards for programmatic access. |
| 88-94 | **Service Continuity Enforcement**: Implements an "Immediate Suspension" policy. When a subscription is deleted at the source (Stripe), the controller updates the tenant status to 'Suspended'. This automated safeguard protects the platform from revenue leakage by instantly revoking access permissions for delinquent accounts. |

---

## ClientController (`clientController.js`)

The `ClientController` provides Super Admins with granular control over multi-tenant operations, including configuration, user management, and resource oversight.

### How it Works
It operates as a management API, allowing admins to bridge the gap between high-level business plans and low-level technical infrastructure (like DB configurations or token limits).

### Functionalities and Features
1.  **getInvoices**: Lists all billing historical records for a specific tenant ID.
2.  **getAuditLogs**: Visualizes tenant-specific telemetry events (Traces) as a human-readable audit trail.
3.  **getTenantUsers / addTenantUser / removeTenantUser**: Full CRUD for user management within an organization.
4.  **getTenantResources**: Maps the infrastructure metadata (DB Shards, Gateways) to a status grid for monitoring.
5.  **getTenantConfig / updateTenantConfig**: Exposes tenant settings as an editable list of environment variables.
6.  **getBenchmarks**: Fetches performance benchmarks associated with a client's specific industry or tier.
7.  **updateSubscription**: Manually modifies a client's active plan.
8.  **addTokenTopUp**: Injects additional AI token capacity into a tenant's quota.
9.  **updateTokenPolicy**: Updates the JSON governing how AI tokens are prioritized or restricted.
10. **updateClient**: A generic bulk update endpoint for primary tenant metadata and settings.

### Feature Logic Descriptions
-   **Config Transformation**: `getTenantConfig` converts a nested JSON `settings` object into an array of `{key, value}` objects. This specialized transformation allows the frontend to render "Environment Variable" style editors without needing to know the underlying JSON structure.
-   **Security Boundaries**: Most endpoints (e.g., `getInvoices`) include a check: `req.tenantId !== id && req.user.role !== 'Super Admin'`. This ensures that a tenant can only see their own data, while Super Admins maintain global oversight.
-   **Dynamic Update Clause**: `updateClient` uses a dynamic SQL query builder. It maps the keys of the `updates` object to a parameterized `SET` clause, allowing for flexible partial updates without multiple trips to the database.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 6-10 | **Multi-Tenant Security Barrier**: Implements a strict access control check. The logic compares the requested ID with the user's active `tenantId` and verifies a 'Super Admin' role. This "Defense in Depth" ensures that even if a regular user discovers another client's ID, the controller will block unauthorized access to sensitive billing and telemetry data. |
| 25-35 | **Granular Telemetry Filtering**: Fetches `telemetry_events` specifically filtered by the target client's ID. This logic transforms global system traces into a tenant-specific audit trail, allowing admins to debug isolated client issues without sifting through platform-wide noise. |
| 98-102 | **Infrastructure Status Virtualization**: Dynamically constructs a map of the client's resource health. By pulling shard and gateway metadata, the logic provides a virtualized status grid, allowing platform owners to monitor the physical infrastructure health assigned to high-value Enterprise clients. |
| 112-123 | **Configuration Mapping Logic**: Converts the nested JSONB `settings` object into a flat array of key-value pairs. This specialized transformation is designed for the frontend's environment editor, allowing for a dynamic UI that can manage a growing number of client-specific variables without needing a hardcoded schema for every new setting. |
| 139-174 | **Atomic Resource Adjustment**: Uses the `||` JSONB concatenation operator for settings updates and `ai_token_limit + $1` for quota increments. These atomic operations prevent race conditions that could occur if multiple admins attempted to update client configurations simultaneously, ensuring high data integrity. |
| 194-212 | **Dynamic Query Builder**: Implements a flexible metadata update engine. The logic builds a parameterized SQL `SET` clause based on the keys provided in the request body. This "Partial Update" strategy minimizes database bandwidth and reduces the risk of accidentally overwriting unrelated client metadata. |

---

## CustomerController (`customerController.js`)

The `CustomerController` is responsible for building a detailed "Customer 360" view, mapping individual user journeys and behavioral predictions.

### How it Works
It utilizes parallel database queries to merge demographic data, behavioral "hot-storage" events, and marketing campaign interactions. It also triggers real-time calculations from the `predictiveService` to provide forward-looking insights like churn risk.

### Functionalities and Features
1.  **getCustomerProfile**: A master aggregator that delivers a complete profile, including basic info, a chronological timeline of interactions, and AI-predicted outcomes.

### Feature Logic Descriptions
-   **Timeline Merging**: The controller fetches events from two different tables (`customer_events` and `campaign_events`). It then standardizes these different schemas into a common `Timeline` object and performs a client-side sort: `.sort((a, b) => new Date(b.date) - new Date(a.date))`.
-   **Predictive Assessment**: It uses `Promise.all` to concurrently invoke churn and LTV assessments. These assessments are processed by the `predictiveService`, which typically uses a weighted scoring model based on the last 30 days of behavior.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 5-13 | **Unified Profile Aggregation**: The `getCustomerProfile` method acts as a high-trust proxy for individual user data. It initializes the response by fetching core demographic markers, providing the identity context necessary for all subsequent behavioral and predictive analysis. |
| 22-37 | **Multi-Source Signal Capture**: Retrieves both "Hot" behavioral signals (customer events) and "Marketing" touchpoints (campaign events). This dual-query strategy ensures that the resulting profile captures the full spectrum of user interaction, from active app usage to passive email engagement. |
| 40-43 | **Predictive Intelligence Integration**: Dispatches concurrent requests to the `predictiveService` to calculate churn risk and LTV (Lifetime Value). This parallel execution ensures that complex AI-driven forecasts are ready by the time the data normalization logic is complete, minimizing total response time for the agent-analyst view. |
| 45-64 | **Chronological Signal Normalization**: Implements a complex merging and sorting algorithm. It standardizes different event schemas into a unified `Timeline` object and applies a descending temporal sort. This logic is why the UI can render a seamless, scrollable "Story of the Customer" regardless of whether the original source was a clickstream or an email open. |

---

## DashboardController (`dashboardController.js`)

The `DashboardController` serves as the technical telemetry engine for the "Observability" and "Support" modules of the platform.

### How it Works
It performs deep-dive aggregations into technical and operational tables. A key architectural feature is its robust fallback system—it detects if a client has no real telemetry data and provides high-fidelity mock data to maintain dashboard utility during initial onboarding.

### Functionalities and Features
1.  **getDatabaseMetrics**: Provides real-time health for technical integrations, including connection pool status and slow query impact analysis.
2.  **getMessagingMetrics**: Tracks the efficiency of conversational AI, measuring deflection rates and human vs. bot handling ratios.
3.  **getSupportMetrics**: Aggregates CSAT (Customer Satisfaction) scores and individual agent handle times.
4.  **getAppStoreMetrics**: Monitors mobile app vitality, including star distributions and crash-free rates.
5.  **getExecutiveConfig**: Fetches the strategic summary configuration (Health Score + Narrative) for the executive view.

### Feature Logic Descriptions
-   **Sub-Query Aggregation**: In `getDatabaseMetrics`, the controller uses advanced SQL with `json_agg` and `FILTER` clauses. This allows it to fetch general metrics, a query heatmap, and a list of slow queries in a single, highly efficient database round-trip.
-   **Deflection Logic**: Deflection is calculated as the ratio of 'Bot Handled' to 'Total Volume'. This provides a direct measure of how much workload the AI agent is successfully offloading from human staff.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 12-38 | **Deep Telemetry Aggregation**: Orchestrates the retrieval of technical health metrics. The logic uses advanced PostgreSQL features like `json_agg` with `FILTER` clauses to fetch main metrics, heatmap nodes, and slow queries in a single atomic transaction. This "Fat Query" approach minimizes the number of database connections required, significantly improving performance for the high-frequency Infrastructure Dashboard. |
| 41-61 | **High-Fidelity Onboarding Safety**: Implements a robust "Mock-Fallback" mechanism. If the database returns no real data for an integration, the controller generates a rich set of realistic mock metrics in JavaScript. This ensures that new users have a visually complete "Day 1" experience while their real telemetry is still being initialized in the background. |
| 85-117 | **Conversational Efficiency Analysis**: Fetches messaging metrics to evaluate bot-human collaboration. The logic calculates deflection rates by comparing AI-handled volume against total requests. By using `Array.from` to generate a 24-hour baseline, the controller ensures that the volume charts are always fully populated even during periods of low traffic. |
| 147-204 | **Agentic Performance Synthesis**: Aggregates satisfaction and resolution data for support teams. It calculates the average handle time (AHT) per agent and generates an hourly sentiment trendline. This logic translates raw event logs into actionable management insights, helping platform owners identify top-performing human agents and surface bottlenecks. |
| 216-285 | **Channel Health Monitoring**: Monitors mobile app store vitality and high-level executive configurations. The logic maps rating distributions and adoption percentages, providing a macro view of platform health for stakeholders. This serves as the "Decision Support" layer, summarizing thousands of data points into a single Health Score and human-readable narrative. |

---

## DataController (`dataController.js`)

The `DataController` represents the "Autonomous Analysis" layer of the platform, where AI agents interact directly with database schemas.

### How it Works
It serves as a secure bridge between an LLM agent (`DataAnalystAgent`) and the raw database. It provides the agent with metadata (schema), receives logic (SQL), and executes that logic in a sanitized, read-only environment via the `analyticsService`.

### Functionalities and Features
1.  **exploreData**: Dispatches natural language data exploration queries. It handles the full loop: Metadata Provision -> Agent Inference -> Secure Execution -> Result Analysis.

### Feature Logic Descriptions
-   **Schema-Aware Inference**: Before dispatching to the agent, the controller fetches the current tenant's database schema. This "Grounding" ensures the agent generates valid SQL that matches the actual table structures.
-   **Safe Execution Proxy**: The controller does not execute SQL directly. It passes the generated query to `analyticsService.executeRawSql`, which is responsible for enforcing read-only constraints and preventing SQL injection or destructive commands.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 5-12 | **Contextual Grounding Strategy**: Before dispatching to the LLM agent, the logic fetches the tenant's current database schema from the `analyticsService`. This "Contextual Grounding" is crucial; it ensures the AI agent understands exactly what tables and columns are available, preventing it from hallucinating non-existent data points in the subsequent exploratory query. |
| 15-22 | **Autonomous Inference Exchange**: Dispatches the user's natural language question along with the schema context to the `DataAnalystAgent`. The logic waits for the agent to return a valid SQL string and a human-readable explanation, effectively outsourcing the "Thinking" part of data exploration to the agent swarm. |
| 26-35 | **Sanitized Execution Proxy**: Implements a strict security protocol for raw SQL execution. The controller passes the agent-generated query to a specialized service that enforces read-only constraints. The logic includes `try-catch` blocks to capture syntax errors, allowing for future "Self-Correction" loops where the agent could attempt to fix its own failing SQL. |
| 39-43 | **Intelligent Insight Delivery**: Assembles the agent's explanation, the verified SQL, and the actual raw data rows into a "Smart Result." This multi-part response allows the frontend to show the "Why" (explanation), the "How" (SQL), and the "What" (Data) simultaneously, providing a high level of transparency and trust for the data exploration process. |
---

## IngestController (`IngestController.js`)

The `IngestController` serves as the high-availability gateway for incoming raw signals. It acts as the first line of defense, validating that every piece of external data conforms to the platform's global schema before it enters the processing pipeline.

### How it Works
It utilizes the **Zod** library for rigorous runtime schema validation. Once a signal is validated, it is passed to the `SignalService` (built using Hexagonal Architecture principles), which handles the transition of the data from its raw state to a "Processing" status where the Agentic Swarm can pick it up.

### Functionalities and Features

1.  **Signal Normalization**: Ensures that content from diverse sources (Shopify, Zendesk, Custom API) is normalized into a standard "Signal" entity.
2.  **Metadata Enrichment**: Accepts optional metadata records that help downstream agents understand the context of the signal.
3.  **Asynchronous Handover**: Returns a 201 status immediately with a `processing` status, indicating that the orchestrator has queued the signal for analysis.

### Feature Logic Descriptions

-   **Input Validation**: The `IngestBodySchema` enforces that every request has valid content and a recognized source. This prevents "Garbage In, Garbage Out" scenarios and protects the backend from malformed data.
-   **Service Delegation**: By calling `this.signalService.ingestSignal`, the controller adheres to the separation of concerns, leaving business logic and persistence to the service layer while it focuses on the HTTP interface.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 1-3 | **Domain Initialization**: Imports Zod for validation and the internal `SignalService`. The dependency on the domain entity `SignalSourceSchema` ensures that the API only accepts sources that the platform is actually capable of processing. |
| 5-9 | **Schema Definition**: Defines the `IngestBodySchema`. The reason for using `z.record` for metadata is flexibility; it allows clients to send any key-value pairs that might be relevant for AI analysis without breaking the ingestion script. |
| 10-14 | **Dependency Injection**: The class constructor accepts the `SignalService`. This enables easier unit testing by allowing developers to mock the service when testing the controller's validation logic. |
| 18-33 | **Ingestion Lifecycle**: The `ingest` method executes a three-step dance: 1) Validate the body against the Zod schema. 2) Hand off to the service for persistence. 3) Return a structured response with a 'processing' status, setting expectations for the asynchronous nature of the AI platform. |
| 34-42 | **Typed Error Handling**: Implements specific catch logic for `ZodError`. This allows the platform to provide detailed "400 Bad Request" messages describing exactly which field failed validation, while catching generic 500 errors for internal system failures. |

---

## NexusController (`NexusController.js`)

The `NexusController` provides the intelligence behind the platform's "Topology" view. it visualizes the live connections between external integrations, internal processing engines, and output buses.

### How it Works
It dynamically queries the `integrations` table to find all active external connections for a tenant. It then "stiches" these dynamic nodes together with a set of standard system infrastructure nodes (like the DLP Gateway and Gemini Cortex) to create a complete graph of the data landscape.

### Functionalities and Features

1.  **Dynamic Topology Generation**: Builds a real-time list of "Source Nodes" based on the actual integrations a user has connected.
2.  **Infrastructure Visualization**: Injects static nodes representing the platform's "Internal Brain" and processing modules.
3.  **Status and Load Mapping**: Assigns health statuses and simulated load metrics to nodes to provide a "live" feel to the network map.
4.  **Provider Iconography**: Maps integration providers (e.g., Slack, Gmail) to specific UI icons for visualization.

### Feature Logic Descriptions

-   **Topology Stitching**: The logic merges `sourceNodes` (dynamic) with `systemNodes` (static). The reason for this is to show users that their data doesn't just "exist"—it flows through established security gates (DLP) and reasoning engines (Cortex).
-   **Icon Context**: The `getProviderIcon` helper ensures that the complex network map is semi-readable at a glance by providing familiar visual cues for different data sources.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 1-3 | **State Access**: Imports the standard database query helper. The controller is designed as a functional object (`NexusController`) rather than a class, favoring a simpler export pattern for this specific view engine. |
| 8-18 | **Active Resource Discovery**: Queries the `integrations` table for non-archived items belonging to the current tenant. This ensures the topology only shows relevant, active data paths. |
| 21-35 | **Resource Node Transformation**: Maps integration rows into a "NexusNode" format. The `load` metric is currently randomized but designed to be replaced by actual data from `sync_history`, providing a visual indicator of data pressure in the system. |
| 39-67 | **Infrastructure Blueprint**: Defines the platform's core "Processing Highway." These nodes are returned for every tenant because they represent the non-negotiable security and intelligence layers (DLP, Gemini Cortex) that all signals pass through. |
| 70-72 | **Graph Assembly**: Merges the dynamic source nodes with the static system nodes. This combined array allows the frontend force-layout engine to render a complete interconnected map of the tenant's intelligence network. |

---

## distributionController (`distributionController.js`)

The `distributionController` handles the "Egress" or delivery phase of surveys and campaigns. It is the bridge between a created research instrument (like a survey) and the customers who need to receive it.

### How it Works
The controller acts as a thin orchestration layer over the `CampaignService`. It translates high-level requests (e.g., "Send this survey to these 100 people via Email") into structured campaigns that the platform's delivery engines (Email/SMS buses) can execute.

### Functionalities and Features

1.  **Survey Distribution**: A specialized endpoint for launching survey-centric campaigns across multiple channels (Email/SMS).
2.  **Bulk Campaign Creation**: A generic endpoint for launching any type of customer engagement campaign.
3.  **Audience Targeting**: Accepts an array of recipients and maps them to a specific campaign instance.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 1-2 | **Service Layer Integration**: Imports the `CampaignService` and the platform logger. Delegating to a service allows the distribution logic to be reused by background chron jobs or internal agents. |
| 7-19 | **Survey Delivery Orchestration**: The `distributeSurvey` function creates a specialized campaign type (`survey_distribution`). It provides a default "Survey Distribution" name if one isn't provided, ensuring the mission is traceable in the admin logs even with minimal input. |
| 21-30 | **Feedback Loop**: Returns the campaign ID and status immediately. If the operation fails, it uses the centralized logger to record the "Survey Distribution Error," which is critical for support teams to debug failed delivery missions. |
| 36-51 | **Generic Campaign Launch**: `createCampaign` allows for high-flexibility campaigns that aren't strictly survey-based. It simply passes the entire request body to the service, enabling forward-compatibility with future campaign types like "Promotional" or "Retention." |
| 56-61 | **Egress Visibility**: `listCampaigns` provides the tenant with a list of all historical and active delivery missions, allowing for the tracking of ongoing distributions. |

---

## finopsController (`finopsController.js`)

The `finopsController` provides technical and financial metrics for AI operations, focusing on the cost-to-value ratio of the platform.

### How it Works
It interfaces with the `CostAttributionService` to calculate profitability and the `ForecastingEngine` to run AI-driven cost simulations. It helps admins understand how business growth translates into infrastructure spend.

### Functionalities and Features

1.  **Profitability Reporting**: Aggregates infrastructure costs (AI tokens, DB shards) against estimated revenue.
2.  **On-Demand Recalculation**: The `raw` query parameter allows admins to force a fresh cost calculation for the current day.
3.  **Causal Forecasting**: Simulates the financial impact of business growth using a predictive engine.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 1-4 | **Financial Services Initialization**: Imports cost attribution logic. This controller separates "spending" (FinOps) from "performance" (Analytics) to provide clear boundaries for platform administrators. |
| 9-15 | **Profitability Trace**: If the `raw` flag is set, it triggers an immediate `calculateDailyCost`. This is useful for real-time spend monitoring. The resulting report gives a high-level view of whether a tenant's usage is economically sustainable for the platform. |
| 24-43 | **Growth Scenario Construction**: The `runForecast` endpoint accepts a `growthRate`. It constructs a "Scenario" by combining historical cost data with an adjustment object that describes the business growth. This maps a business metric (Growth %) to a technical impact (Cost Change). |
| 46-51 | **Predictive Simulation**: Calls the `forecastingEngine.simulate`. This logic doesn't just "multiply" costs; it uses an AI model to predict how data volume and token usage scale non-linearly with user growth. |
| 52-58 | **Strategic Projections**: Returns a full forecast series for charting. The message included in the response is designed to be user-facing, explaining exactly how the AI projection arrived at the "Projected Cost." |

---

## governanceController (`governanceController.js`)

The `governanceController` handles the "Safe Boundaries" of the platform. It manages communication policies, audit logs of traffic, and customer fatigue monitoring.

### How it Works
It manages a suite of `governance_policies` which are rules that govern how the platform interacts with customers. It also monitors "Fatigue," which is a weighted measure of how saturated a customer segment is with messages, suggesting pauses to prevent churn.

### Functionalities and Features

1.  **Policy CRUD**: Handles the lifecycle of automated governance rules (e.g., "Don't message VIPs more than once a day").
2.  **Traffic Audit**: Provides visibility into the `governance_logs`, which record every time a policy blocked or allowed a signal.
3.  **Fatigue Assessment**: Analyzes segments of the customer base to determine their risk of "burnout" from over-communication.
4.  **Recommended Mitigation**: Provides AI-generated suggestions (e.g., "Pause all comms for 7 days") based on fatigue levels.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 2-3 | **Compliance Infrastructure**: Imports the database helper and logger. Governance requires high traceability, so the logger is used extensively for all policy-related actions. |
| 5-18 | **Policy Catalog Retrieval**: Fetches all policies for a tenant. It maps DB rows to a clean frontend object, ensuring that the `aiConfidence` and `stats` (blocked/override counts) are easily accessible for the Governance Dashboard. |
| 26-36 | **Policy Definition**: `createPolicy` allows admins to insert new rules into the system. This logic persists the "Logic" field, which is often a JSON string or a simple rule-set that the Ingestion Bus executes at runtime. |
| 58-75 | **Live Traffic Auditing**: Retrieves the most recent 50 logs of governed traffic. The logic formats the ISO timestamp into a human-readable local time, allowing admins to quickly scan for recent policy violations or blocks. |
| 84-107 | **Segment Burnout Analysis**: The `getFatigueSegments` logic is the core of the platform's "Customer Empathy" engine. It orders segments by `risk_level` so that the most critical issues appear at the top. If no data exists, it returns high-fidelity mock data to ensure the admin understands the value of the dashboard before real events are tracked. |

---

## healthController (`healthController.js`)

The `healthController` is the "Vital Signs" monitor of the platform. It provides real-time visibility into the operational state of both technical infrastructure and background processing layers.

### How it Works
It executes active probes against core dependencies. By timing a "SELECT 1" query and measuring response times for internal process lookups, it creates a high-fidelity snapshot of platform latency and resource availability without impacting production performance.

### Functionalities and Features

1.  **Multi-Dimensional Health Check**: Monitors Database, Redis, Queues, and the API Gateway in a single unified response.
2.  **Resource Diagnostics**: Exposes process-level metrics including memory heap usage and system uptime.
3.  **Graceful Degradation Detection**: Automatically flags the platform as "Degraded" if a non-critical service (like a background queue) fails, while still allowing the API to serve requests.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 6-15 | **Health State Initialization**: Defines the baseline "Operational" state and initializes a structured map for all core services. This proactive structure ensures that even if a check fails early, the response maintains a consistent JSON schema for the monitoring dashboard. |
| 23-32 | **Database Latency Probe**: Executes a minimal query and calculates the delta in milliseconds. The logic updates the database status to 'Critical' if the probe fails, providing immediate feedback for infrastructure auto-scaling or incident management. |
| 35-56 | **Infrastructure Continuity Check**: Attempts to verify connectivity to Redis and the `QueueService`. This logic is critical for identifying "Silent Failures" where the web server is fine, but background jobs (like email delivery) are stalled. |
| 58 | **Unified Health Export**: Returns the complete diagnostic object. The timestamp in the response ensures that monitoring agents can verify they are looking at "Fresh" data, preventing stale-cache issues during outages. |

---

## hierarchyController (`hierarchyController.js`)

The `hierarchyController` manages the organizational "Backbone" of a tenant. It allows for the modeling of complex regional, departmental, and team structures.

### How it Works
It provides a flexible interface for managing `org_units`. It supports both granular subtree retrieval (for multi-level navigation) and atomic full-tree replacement, ensuring that organizational changes are handled with high integrity and a clean audit tail.

### Functionalities and Features

1.  **Contextual Tree Navigation**: Allows for the retrieval of specific sub-orgs via `rootId`, facilitating "Drill-Down" UI patterns.
2.  **Atomic Structure Updates**: Implements a "Delete-and-Rebuild" strategy for organizational changes, ensuring no orphaned nodes remain during complex restructures.
3.  **Traceable Re-orgs**: Every major hierarchy update is recorded in the `auditService`, capturing exactly which administrator modified the organizational chart and the scale of the change.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 6-21 | **Intelligent Hierarchy Fetching**: Determines whether to return a specific branch or the entire organizational forest. It uses the `hierarchyService` for complex subtree calculations, maintaining a thin controller layer that focuses on request routing. |
| 33-44 | **Atomic Org Restructuring**: Executes a destructive but safe rewrite of the organization tree. By deleting and re-inserting all nodes in a single flow, it avoids the "Ghost Node" problem often found in incremental tree updates, ensuring the resulting Org Chart matches the administrator's exact intent. |
| 47-55 | **Compliance and Narrative Audit**: Logs the `HIERARCHY_FULL_UPDATE` action. The reason for including the `nodeCount` in the audit details is to provide a quantitative marker of the re-org's scale for future compliance reviews. |
| 62-80 | **Topographical Statistics**: Aggregates node counts by type (e.g., 'Region', 'Store'). This logic provides a high-level summary of the organization's breadth, helping admins verify that their digital twin structure matches their physical business footprint. |

---

## ingestionController (`ingestionController.js`)

The `ingestionController` provides a dedicated entry point for manual data imports, complementing the automated API-based signals handled by the `IngestController`.

### How it Works
It utilizes the **Multer** middleware to handle multi-part form data (file uploads). It strictly enforces a 10MB limit to protect server memory and uses a memory-storage strategy to process files without needing to write sensitive data to the disk, fulfilling strict privacy requirements.

### Functionalities and Features

1.  **Format-Agnostic Uploads**: Supports manual imports of CSV, Excel (XLSX), and JSON files.
2.  **Stateful Import Modes**: Allows users to choose between 'Append' (adding to existing data) or 'Replace' (nuking old records for that source), providing full control over data updates.
3.  **Memory-Safe Processing**: Processes binary streams in-memory, ensuring that customer data never touches the server's persistent storage during the conversion phase.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 5-8 | **Ingestion Guardrails**: Configures Multer with memory storage and a 10MB file cap. These "Hard Limits" are essential security measures to prevent Denial of Service (DoS) attacks through massive file uploads. |
| 36-42 | **Input Integrity Validation**: Verifies that both a file and a valid `source` (e.g., 'Facebook Offline') are present. This ensures that every manual import has an "Owned By" tag, maintaining data provenance within the platform. |
| 43-50 | **Service-Layer Delegation**: Hands the file buffer off to the `fileUploadService`. By separating the "Receiving" logic from the "Parsing" logic, the controller remains responsive while complex file transformations happen in the background. |

---

## integrationController (`integrationController.js`)

The `integrationController` is the core configuration engine for the platform's multi-source connector ecosystem. It manages the lifecycle of 80+ third-party integrations.

### How it Works
It acts as a data-merging engine. It pulls the list of connected integrations from the database and merges them with "Templates" from the `IntegrationCatalog`. This architecture allows the platform to support a massive library of providers while only storing the tenant-specific secrets and mappings in the database.

### Functionalities and Features

1.  **Catalog-Driven Discovery**: Merges active connections with the global catalog to show available vs. connected providers.
2.  **Secure Credential Management**: Implements a "Double-Safety" encryption strategy, using specific sanitization for copy-paste errors and hex-encoded encryption for storage.
3.  **Privacy-First Deletion (Nuke)**: Provides an atomic "Nuke" endpoint that deletes configuration, sync history, and all associated raw events to comply with GDPR/CCPA.
4.  **Schema Mapping Engine**: Manages the logic that translates external fields (like 'user_id') into platform-internal identities (like 'customer_id').

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 10-19 | **Active Connection Mapping**: Fetches the tenant's current integration state. By converting the database rows into a `Map`, the logic enables O(1) lookups during the subsequent catalog merge, ensuring the integrations page loads instantly regardless of the number of connectors. |
| 27-63 | **Catalog Synthesis Strategy**: Iterates through the global `IntegrationCatalog`. This logic is responsible for "Upgrading" legacy connections by injecting default schema mappings if they are missing from the database, ensuring forward-compatibility with new platform features. |
| 82-122 | **Safe Disconnection & Purge**: Orchestrates the removal of an integration. The reason for the complex `DELETE` query on lines 106-114 is to ensure that even legacy data (missing `integration_id`) is scrubbed based on its `source` tag, fulfilling the platform's "Right To Be Forgotten" obligation. |
| 131-153 | **Cryptographic Secret Hardening**: Implements a dedicated encryption loop for sensitive keys. It specifically avoids re-encrypting already-secured values, preventing data corruption during configuration updates while ensuring all tokens are unreadable at rest. |
| 242-300 | **Adaptive Credential Sanitization**: Implements a "Deep Cleaning" phase for API keys. It handles common user errors like accidental quotes, newlines, or even copy-pasting an entire JSON object by mistake. This logic reduces support tickets by "Making things just work" for the end-user. |
| 302-459 | **Facebook Graph API Validation**: A specialized, high-rigor validation flow for Meta integrations. It includes a `fetchWithRetry` helper with exponential backoff to handle Facebook's occasionally flaky API, ensuring a robust connection experience. |
| 656-702 | **Atomic Platform Reset (Nuke)**: Provides the ultimate privacy tool. It orchestrates a three-stage delete (Events -> History -> Config). This ensures that a tenant can completely "Reset" a channel if they suspect data contamination or need to comply with a legal request. |

---

## intelligenceController (`intelligenceController.js`)

The `intelligenceController` is the "Thinking" layer of the VoC platform. it manages the lifecycle of customer behavioral models and strategic simulations.

### How it Works
It orchestrates interactions between various specialized services like the `TwinEvolutionService` and the `EntropyScanner`. It manages the creation of "Digital Twins"—AI models that represent real customer segments—and allows admins to test business decisions against these models through wargaming.

### Functionalities and Features

1.  **Digital Twin Provisioning**: Creates and maintains psychographic profiles for specific account IDs.
2.  **Scenario Wargaming**: Runs simulations to predict how a digital twin will react to specific business changes (e.g., "Change to 3-year contract").
3.  **Swarm Intelligence Simulation**: Conducts "Virtual Focus Groups" by running a topic through a swarm of multiple digital twins.
4.  **Prescient Risk Scanning**: Uses the `EntropyScanner` to identify customer segments that are trending toward chaotic behavior or high churn risk.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 15-34 | **Model Construction**: Triggers the `customerModelService` to build a twin. This logic initializes the "Intelligence" for an account, synthesizing historical events into a behavioral profile that subsequent simulations can reference. |
| 41-59 | **Behavioral Wargaming**: Dispatches a "Price Increase" or "Policy Change" scenario to the twin. The logic returns the twin's most likely reaction, allowing business owners to preview the impact of unpopular decisions before they are launched to real people. |
| 86-97 | **Aggregated Social Simulation**: The `runSwarm` method scales intelligence by simulating a focus group. It allows for "Market Testing" a topic across a balanced segment of digital twins, providing a probabilistic outcome for platform-wide changes. |
| 103-111 | **Chaos Prevention Engine**: Initiates an "Entropy Scan." The reasoning behind this endpoint is to provide a "Proactive Alerting" system that identifies early signals of behavioral decay before they manifest as hard metrics like Churn. |
| 55-68 | **Mobile Workflow Interface**: Fetches `mobile_drafts` with a strict filter on 'pending' status. The reasoning for this endpoint is to enable "One-Tap Approvals" on the go, allowing admins to clear their AI-response queue from their phone. |

---

## notificationController (`notificationController.js`)

The `notificationController` manages the delivery and state of tenant-wide alerts and system-generated notifications.

### How it Works
It leverages the `alerts` table as a dual-purpose store for both technical system warnings and user-facing notifications. This unified approach simplifies the frontend state management, allowing a single component to handle all real-time messaging.

### Functionalities and Features

1.  **Context-Aware Alert Fetching**: Retrieves a rolling window of the most recent 50 alerts, mapping their "Severity" (Critical/Info) to visual UI types.
2.  **Bulk State Management**: Provides an "Archive All" equivalent via the `markAllAsRead` endpoint, reducing database write-pressure during large-scale alert events.
3.  **Traceable Event Removal**: Ensures that users can clean their workspace while maintaining the original audit trail in the backend logs.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 5-23 | **Alert Normalization**: Standardizes raw database rows into a "Notification" object. The reason for the `toLocaleTimeString` conversion on line 18 is to ensure that relative "Time Since" calculations in the UI are grounded in a valid ISO-compatible string. |
| 30-41 | **Granular State Update**: Updates the `is_read` flag for a specific record. The logic includes a strict `tenant_id` check, preventing cross-tenant data manipulation even if an alert ID is guessed. |
| 44-55 | **Atomic Bulk Read**: Executes a single `UPDATE` query for all of a tenant's alerts. This is a high-efficiency operation designed to clear the notification badge in one transaction. |

---

## moduleController (`moduleController.js`)

The `moduleController` is the primary orchestrator for the platform's specialized "Business Case" modules, defining the behavior of the Agentic Swarm.

### How it Works
It utilizes a **Dynamic Factory** pattern to instantiate specialized `VoCModuleAgent` brains (e.g., "Loyalty Analyst", "Product Gap Hunter") based on the module's ID. It acts as the bridge between high-level business goals (defined in the `BUSINESS_CASES` config) and their executable AI counterparts.

### Functionalities and Features

1.  **AI Brain Awakening**: Initializes and hydrates a module's agent, creating an in-memory presence that is ready to process signals.
2.  **Predictive Impact Simulation**: Allows admins to preview how an agent will react to hypothetical customer signals before the agent is allowed to execute real-world actions.
3.  **Cross-Pillar Impact Reporting**: Merges hardcoded configuration with live database status to show the tenant exactly which "Intelligence Modules" are active and what metrics they are optimizing (Revenue, Risk, etc.).
4.  **Autonomous Action Delegation**: Hands off confirmed agent decisions (like "Send Apology") to the `ActionExecutorService` for real-world fulfillment.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 10-60 | **Intelligent Agent Factory**: This is the controller's "Core Logic." It maps a `moduleId` to a specific identity and goal. The reasoning for including `killPaths` (actions the agent is allowed to take) ensures that the AI is constrained by "Guardrails" from the moment it is created. |
| 62-72 | **Agent Context Re-hydration**: The `awakenAgent` method calls `agent.awaken()`. This logic is responsible for verifying that the agent's internal state is healthy and that its "Memory" of the tenant's past data is loaded. |
| 115-150 | **Dynamic Configuration Synthesis**: Merges the static `BUSINESS_CASES` with the `modules` table. This allows the platform to offer "New Modules" through code updates while allowing tenants to maintain their own "On/Off" toggle and custom configurations in the DB. |
| 197-208 | **Deep-Dive Synthesis**: Triggers a high-fidelity report generation. The agent uses its "Module Goal" to filter through recent signals and find the most meaningful patterns, effectively creating an on-demand consultant-grade presentation. |

---

## omniController (`omniController.js`)

The `omniController` provides the multi-channel "Omni-Query" interface, allowing users to converse with their data via web search or external apps like Slack.

### How it Works
It serves as an ingestion wrapper for the `NaturalQueryEngine`. It handles the transition from "Raw User Input" to "Structured AI Query," specifically managing the asynchronous feedback loops required for third-party platforms with strict response timeouts (like Slack).

### Functionalities and Features

1.  **"Talk to Data" Interface**: The primary web portal for natural language research across the entire tenant's data lake.
2.  **Slack Integration Hub**: Implements the Slack Slash Command protocol, providing immediate "Thinking" feedback while processing complex queries in the background.
3.  **Omni-Channel Research Library**: Retrieves files, transcripts, and documents that have been ingested through the platform's multi-source buses.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 9-20 | **Direct Inference Gateway**: Passes a raw string to the `naturalQueryEngine`. The logic is intentionally minimal to ensure that the engine's "Self-Correction" and "Refining" capabilities are not blocked by controller-level validation. |
| 27-51 | **Slack Protocol Compliance**: Implements the "Double Response" pattern. Slack requires a response within 3 seconds, so the logic immediately returns a "Thinking" message and spawns a background `_processAsyncSlack` call to handle the actual AI work. |
| 53-84 | **Async Result Rendering**: Executes the query and formats the result as "Slack Blocks." This logic is responsible for taking raw data and turning it into human-readable Slack markdown, including a snippet of the found records for validation. |
| 86-110 | **Media Context Retrieval**: Fetches file metadata from the `omni_files` table. The `timeline` array inclusion ensures the frontend can render "Heatmaps" of events that occurred within a specific recording or document. |

---

## performanceController (`performanceController.js`)

The `performanceController` provides the infrastructure for "Self-Healing" and data-driven optimization of the platform's API layer.

### How it Works
It exposes the internal telemetry captured by the `performanceLogger`. It doesn't just show metrics; it correlates them with active "Optimization Rules," identifying which slow endpoints are already being mitigated and which needs a new cache-rule or worker-scale boost.

### Functionalities and Features

1.  **Endpoint Health Dashboard**: Categorizes every API route by latency and error rate (Critical/Warning/Healthy).
2.  **Auto-Optimization Discovery**: Flags endpoints that are missing cache rules, providing a direct link to the "Fix" mechanism.
3.  **Observability-Driven Repair**: Allows admins to trigger a `triggerOptimization` call, which uses the `OptimizerService` to apply technical fixes (like adding an N+1 query guard or a cache rule) without a code deployment.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 4-34 | **Telemetry Transformation**: Iterates through the raw endpoint logs. The logic calculates the average latency and compares it against "Health Threshholds" (e.g., <200ms is Healthy). This provides an instant "Heatmap" of the platform's performance bottlenecks. |
| 21 | **Optimization Correlation**: Checks if an endpoint's route is currently covered by a `cacheRule`. This provides immediate proof-of-value for the platform's auto-scaling and caching services. |
| 36-44 | **Service-Level Self-Repair**: Dispatches a fix request to the `optimizer`. By specifying a `type` and `target`, the admin can "Inject" new efficiency rules into the running system, effectively tuning the platform in real-time. |

---

## productGraphController (`productGraphController.js`)

The `productGraphController` manages the relational "Ontology" of a tenant—the mapping of products to their features and the users who interact with them.

### How it Works
It maintains a strictly hierarchical view of data. By organizing everything around a "Product" root, it enables the "Revenue Attribution" engine to correctly identify which feature is driving the most financial value or causing the most churn risk.

### Functionalities and Features

1.  **Feature-Product Mapping**: Manages the CRUD lifecycle of the entire product catalog for a tenant.
2.  **Account Usage Context**: Tracks which specific customer accounts are associated with which product instances.
3.  **Value Attribution Linking**: Provides the `linkAttribution` endpoint, which maps platform-level events to specific features for "Business Value" calculations.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 7-15 | **Contextual Product Listing**: Retrieves all products for a tenant. The logic is thin, delegating to the `ProductGraphService` to handle any necessary joins with Feature counts or Account summaries. |
| 41-48 | **Featurization Workflow**: Automatically links a new feature to its parent `productId`. This hierarchy is essentially the "Skeleton" that the AI uses when it needs to understand why a customer is complaining about a specific part of the software. |
| 74-82 | **Attribution Mapping**: Connects an external ID to a specific product/feature node. This logic is the key to "Value Engineering," as it allows the platform to say: "Feature $X$ is mentioned in $Y\%$ of Churn signals." |

---

## reportController (`reportController.js`)

The `reportController` handles the "Storytelling" layer of the platform, enabling users to build and share custom dashboards.

### How it Works
It manages the transition from "Raw Metrics" to "Visual Insights." It stores report configurations as a collection of "Widgets" in a single JSON blob, providing a flexible "Canvas" architecture that can support any type of chart or table the user needs.

### Functionalities and Features

1.  **Collaborative Report Drafting**: Tracks the `status` of a report (Draft/Published) and who created it, enabling enterprise-scale reporting workflows.
2.  **Dynamic Dash Assembly**: Operates as a widget store, allowing the frontend to save complex dashboard layouts (positions, chart types, data sources) in a single update.
3.  **Chronological Versioning**: Automatically updates the `updated_at` field on every save, ensuring the "Recent Reports" view is always accurate.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 5-27 | **Visual Inventory Management**: Fetches a list of reports with their `widgets` configuration. The transformation on line 18 ensures the date is presented in a localized format for easy scanning in the admin interface. |
| 35-47 | **Report Provisioning**: Inserts a new draft report and records the `userId` as the creator. This logic is vital for multi-user accounts where tracking "Who" built a specific dashboard is required for internal accountability. |
| 54-74 | **Widget State Synthesis**: The `updateReport` method accepts a `widgets` array. By converting this to a JSON string on line 67, the logic persists the entire visual state of the dashboard in a single database column, maximizing flexibility for different chart types. |

---

## journeyController (`journeyController.js`)

The `journeyController` manages customer experience pathways, visualized as graph-based workflows of interactions and touchpoints.

### How it Works
It stores complex graph data (Nodes and Edges) as structured JSON. To handle high-concurrency environments (e.g., multiple admins editing the same path), it implements an "Optimistic Locking" mechanism using a `lastUpdatedAt` timestamp check before committing changes.

### Functionalities and Features

1.  **Graph Life-cycle Management**: Standard CRUD for organizational paths, ensuring that "Nodes" (touchpoints) and "Edges" (transitions) are always kept in sync.
2.  **Concurrency Protection**: Prevents "Last-Write-Wins" data loss by verifying the document's version before update.
3.  **Simulated Path Tracing**: A placeholder for AI-driven simulations where a "Virtual Customer" is sent through the journey to find bottlenecks.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 10-21 | **Schema Translation Layer**: Normalizes the raw database JSON into a format the frontend graph engine understands. It explicitly maps `steps` to `nodes` and `edges`, ensuring backward-compatibility with older journey schemas while serving a modern React Flow interface. |
| 35-51 | **Structure Persistence**: Assembles the graph components into a single `steps` blob. The reason for returning the newly created ID and stats immediately is to allow the UI to transition seamlessly from a "Draft" state to a "Live" editing state without a full page refresh. |
| 64-70 | **Optimistic Locking Guard**: Compares the client-provided `lastUpdatedAt` with the database's current state. This logic is crucial for multi-user environments, as it informs the user if a colleague has modified the journey while they were editing, preventing accidental overwrites of complex logic. |

---

## knowledgeController (`knowledgeController.js`)

The `knowledgeController` is the core of the platform's RAG (Retrieval-Augmented Generation) infrastructure, managing the tenant's private document corpus.

### How it Works
It coordinates a three-stage ingestion pipeline: Relational Storage (Postgres), Intelligent Extraction (Librarian Agent), and Vector Indexing (Semantic Search). This ensures that documents are not just stored, but "understood" and searchable by intent rather than just keywords.

### Functionalities and Features

1.  **Intelligent Document Ingestion**: Orchestrates the movement of data from a raw string to an indexed "Entity-Aware" knowledge item.
2.  **Semantic Search**: Interfaces with a `VectorStore` to perform context-aware retrieval across all indexed documentation.
3.  **RAG-Powered Chat**: Dispatches user questions to a "Librarian Agent," which synthesizes answers using only the verified private knowledge bank.
4.  **Knowledge Base Audit**: Uses AI to identify "Gaps" (e.g., questions asked by users that aren't answered in the docs).

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 37-48 | **Ingestion Orchestration**: First persists metadata to ensure the user sees an immediate "Processing" status, then dispatches to the `Librarian Agent`. This "Multi-Mode" approach ensures that if agent extraction fails, the document index is still initiated, maintaining system reliability. |
| 52-60 | **Dual-Engine Indexing**: Adds the content to the `vectorStore` for semantic search while updating the relational DB with extracted entities. The reason for this dual storage is to support both "Chat" (Semantic) and "Graph Visualization" (Relational) features. |
| 85-98 | **Semantic Retrieval Logic**: Passes the user's query through an embedding engine to find the top $N$ relative snippets. This logic is the engine behind "Smart Search," providing results based on meaning rather than string matching. |
| 123-141 | **Agentic Synthesis**: Dispatches to the `agent_librarian_01`. The logic specifically requests the `sources` used by the AI, which are then shown to the user to build trust and prevent hallucinations in the knowledge-sharing process. |

---

## marketController (`marketController.js`)

The `marketController` provides competitive intelligence and "Wargaming" scenarios to help tenants navigate their industry landscape.

### How it Works
It blends traditional data scraping (Market Signals) with interactive AI role-playing. It enables admins to run simulations where the Gemini AI adopts the persona of a competitor's CEO to test how they might react to a specific business move.

### Functionalities and Features

1.  **Executive Tracker**: Monitors competitor leadership changes and public announcements.
2.  **Scenario Wargaming**: A generative AI role-play environment where admins can test strategic moves (e.g., "Change to usage-based pricing").
3.  **Automated Battlecards**: Uses the `agent_market_01` to synthesize competitive data into a strategic "Cheat Sheet" for sales teams.
4.  **Radar Visualization**: Maps the platform's scoring against competitors across multiple qualitative dimensions.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 28-58 | **Role-Play Persona Injection**: Constructs a high-rigor prompt for Gemini using competitor attributes. The logic specifically instructs the AI to output strictly JSON, ensuring the "Thought Process" and "Win Probability Shift" can be rendered as interactive components in the UI. |
| 62-67 | **Robust JSON Extraction**: Uses a regex-match strategy on the AI's response. Because LLMs sometimes include conversational text before/after JSON, this logic ensures the controller always extracts the structured data, preventing UI crashes. |
| 113-146 | **Snappy Resource Creation**: Creates the competitor record and immediately returns it to the user. The `refreshCompetitor` scan is triggered asynchronously (not awaited), ensuring the UI stays responsive while the expensive background scraping process begins. |
| 235-252 | **Specialized Agent Dispatch**: Outsources the "Battlecard" creation to a dedicated Market Agent. By using an agent instead of a static template, the battlecard can use the absolute latest signals found during the last scan, providing "Real-Time" competitive intelligence. |

---

## mediaController (`mediaController.js`)

The `mediaController` manages the high-bandwidth ingestion of audio and video feedback, such as customer interview recordings or video surveys.

### How it Works
It implements a "Signed URL" pattern to offload the heavy lifting of binary uploads. Instead of sending files through the application server, the controller generates a temporary, secure permission for the client to upload directly to cloud storage (S3/GCS).

### Functionalities and Features

1.  **Cloud-Native Ingestion**: Generates GCS/S3 signed URLs with support for automatic fallback to a "Local Secure Token" for development environments.
2.  **Webhook Completion Handler**: Receives notifications from the cloud provider when an upload is finished, acting as the bridge to the platform's processing queues.
3.  **Tier-Based Prioritization**: Assigns processing priority (VIP vs. Standard) based on the tenant's tier, ensuring Enterprise customers receive faster transcription and analysis.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 14-39 | **Secure Delegation Engine**: Determines whether to use real cloud credentials or a local HMAC-signed fallback. This logic ensures the app is portable across Dev and Prod environments while maintaining the "Upload-to-Cloud" architectural pattern. |
| 41 | **Traceable Logging**: Logs the file size and name before generating the URL. This provides a "Paper Trail" for ingestion attempts, allowing admins to track if large files are being attempted before they even hit storage. |
| 50-68 | **Async Ingestion Handover**: Receives the cloud webhook and dispatches a job to the `MediaQueue`. By passing the `tenantId` and `tier` in the metadata, the logic ensures the neural-processing workers can correctly prioritize the expensive GPU-based analysis. |

---

## mobileController (`mobileController.js`)

The `mobileController` is a specialized optimization layer designed to serve the Loompin VoC mobile application with high-concurrency, low-latency data.

### How it Works
It acts as a "BFF" (Backend for Frontend) specifically for mobile devices. It aggressively uses a "Read-Through" cache to prevent expensive database re-scans for common resources like the activity feed and velocity alerts.

### Functionalities and Features

1.  **Cached Mobile Feed**: Aggregates health alerts, NPS metrics, and recent events into a single query-collapsed response.
2.  **"Summary-Level" Data**: Returns "Lite" versions of customer records to save bandwidth and improve mobile render times.
3.  **Cross-Device Push Registry**: Handles the storage of tokens required for FCM (Firebase) or APNS (Apple) notifications.
4.  **Draft Approval Hub**: Provides a specialized view for "Pending" AI drafts that require manual approval from a mobile device.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 10-41 | **Cache-Wrapped Aggregation**: Wraps the entire feed generation in a 60-second cache. This logic is a critical performance trade-off; it accepts 1-minute staleness in exchange for the ability to handle thousands of concurrent mobile requests without hammering the relational database. |
| 24-27 | **Mobile-Specific NPS Logic**: Executes a specialized aggregate query to calculate a 24-hour sentiment score. This "Lite" aggregation provides the headline metric required for the mobile app's "Morning Briefing" dashboard. |
| 55-68 | **Mobile Workflow Interface**: Fetches `mobile_drafts` with a strict filter on 'pending' status. The reasoning for this endpoint is to enable "One-Tap Approvals" on the go, allowing admins to clear their AI-response queue from their phone. |

---

## projectController (`projectController.js`)

The `projectController` manages the long-term strategic initiatives and their tactical execution via projects and tasks.

### How it Works
It implements a hierarchical management system where "Projects" (high-level goals) are broken down into "Tasks" (atomic actions). The controller includes a discovery layer that uses keyword-based correlation to link raw customer feedback directly to active strategic projects, providing "Live Validation" of the project's impact.

### Functionalities and Features

1.  **Strategic Portfolio Tracking**: Manages the lifecycle of projects, including progress, milestones, and predicted efficiency outcomes.
2.  **Predictive Impact Forecasting**: Provides a 12-month projection of business value vs. a baseline, helping stakeholders visualize the ROI of a specific initiative.
3.  **Customer-Project Correlation**: Automatically identifies customer feedback that matches a project's theme, proving the "Demand" or "Necessity" of the work in real-time.
4.  **Tactical Task Management**: Handles the day-to-day assignment, priority, and due-dates for teams executing the project work.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 10-22 | **Strategic Entity Transformation**: Maps raw database projects into a high-fidelity frontend object. The reason for the `predictedOutcome` placeholder on line 21 is to ensure that every project in the UI has a "Reason for existing" (Value Statement) displayed to stakeholders. |
| 114-139 | **Projective Modeling Engine**: Retrieves or simulates a 12-month impact forecast. If no historical data exists, it uses a linear growth simulation (line 132) to provide a "Visual Goal" for the project, setting expectations for ROI. |
| 141-183 | **Intelligent Feedback Discovery**: Executes a keyword-search against the `events` table (transcripts/surveys). By matching project titles to customer comments, the logic "Grounds" the project in real-world customer needs, effectively automating the "Voice of Customer" linkage. |

---

## pulseController (`pulseController.js`)

The `pulseController` is the engine room of the platform's automated reflex system, enabling the system to react instantly to customer signals.

### How it Works
It manages dynamic "Reflexes" (if-this-then-that triggers) and "Playbooks" (structured response plans). It features a **Reinforcement Learning Loop** where human approvals/rejections of AI-initiated actions are used to tune the system's `confidence_threshold`.

### Functionalities and Features

1.  **AI-Generated Reflexes**: Allows users to describe a trigger in plain English and uses the `pulseAgentService` to generate the formal logic and action plan.
2.  **Human-in-the-Loop Learning**: Implements a learning mechanism where approving an action decreases the future confidence threshold (granting more trust), while rejecting one increases it (becoming more conservative).
3.  **High-Concurrency Pulse Processing**: Orchestrates the parallel evaluation of live signals against all active reflexes, ensuring reactions happen in milliseconds.
4.  **Strategic Playbook Repository**: Manages long-term response strategies for repetitive issues like "Billing Outages" or "NPS Drop."

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 31-61 | **Conversational Logic Provisioning**: Detects if a `prompt` is provided and dispatches it to the AI for structuring. This logic removes the need for users to know "Database Fields" or "Trigger Syntax," making automated governance accessible to business users. |
| 101-133 | **Reinforcement Learning Bridge**: Adjusts the `confidence_threshold` based on user feedback. The reason for the `GREATEST(50, ...)` and `LEAST(99, ...)` constraints is to ensure that the AI never becomes truly fully autonomous (safety floor) or completely disabled. |
| 139-205 | **Parallel Reflex Engine**: The platform's "Central Nervous System." It uses `Promise.all` to evaluate a single signal against every active reflex simultaneously. If a match is found and passes the confidence threshold, it orchestrates the `explainDecision` logic to provide a human-readable "Why" before execution. |

---

## rbacController (`rbacController.js`)

The `rbacController` manages the security and access permissions landscape of the multi-tenant platform.

### How it Works
It operates a stateful Role-Based Access Control system. It uses an **In-Memory Cache** with a 60-second TTL to improve the performance of permission checks, ensuring that checking "Can user X view Dashboard Y" does not require a database hit every time.

### Functionalities and Features

1.  **Dynamic Role Matrix**: Manages a complex mapping of roles (e.g., Brand Admin, Analyst) to specific functional views and edit actions.
2.  **Context-Isolated Cache**: Keys the permission cache by `tenantId`, strictly preventing any cross-tenant leakage of security policies during the TTL window.
3.  **SOC 2 Audit Logging**: Every update to the role matrix is logged, ensuring that all security configuration changes are traceable to a specific administrator for compliance.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 7-40 | **Fallback Security Baseline**: Defines the `DEFAULT_MATRIX`. This logic serves as the "Safety Net" that ensures the platform has a strict, well-known set of permissions even if a tenant hasn't defined their own custom policies yet. |
| 47-93 | **High-Efficiency Role Fetching**: Implements a "Cache-Aside" pattern. It first checks the `matrixCache` (lines 50-55) before falling back to Postgres. This preserves database bandwidth for more expensive analytics queries while keeping security checks fast. |
| 95-124 | **Secure Policy Persistence**: Updates the persistent `rbac_matrix` in `security_policies`. The reasoning for the immediate cache-update on line 114 is to ensure that "Administrative Wins" (e.g., granting a user new access) take effect within milliseconds without waiting for a cache expiry. |

---

## roadmapController (`roadmapController.js`)

The `roadmapController` manages the strategic roadmap of product features, prioritizing them based on their business impact and effort.

### How it Works
It is a **Value-First** management tool. It includes a built-in RICE scorer (Reach, Impact, Confidence, Effort) that automatically calculates a `total_score` whenever a roadmap item is created or modified. This ensures that the roadmap is always driven by data-backed priorities rather than intuition.

### Functionalities and Features

1.  **Automated RICE Scoring**: Validates the components of prioritize work and generates an objective score for every roadmap item.
2.  **Dynamic Item Drafting**: Allows for the flexible creation of roadmap items with strategic themes, ETAs, and dependency mappings.
3.  **Audience Impact Planning**: Tracks the "Strategic Theme" and "Tags" for every item, enabling admins to filter the roadmap by business pillars (e.g., "Retention", "Revenue").

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 23-28 | **Prioritization Logic**: Implements the RICE math formula. By normalizing the confidence as a percentage (line 26), the logic ensures the resulting score is a balanced representation of the item's potential business value vs. risk. |
| 49-93 | **Granular Item Tuning**: A complex dynamic query builder. The reason for recalculating the RICE score on line 65-70 is to ensure that if a product manager increases the "Effort" or decreases the "Confidence," the roadmap ranking is updated instantly in the database. |

---

## roiController (`roiController.js`)

The `roiController` is the financial dashboard for product management, quantifying the dollar-value impact of current and future roadmap work.

### How it Works
It acts as a strategic reporting layer over external tools like Jira. It fetches the latest product work and passes it to the `featureROIService`, which applies "Context Multipliers" (region, industry) to estimate the potential revenue or savings an item will generate.

### Functionalities and Features

1.  **Opportunity Prioritization**: Retrieves a financial ranking of roadmap items, showing exactly which initiatives have the highest `impact_amount`.
2.  **Jira Strategic Sync**: Triggers a deep reconciliation between the development backlog (Jira) and the platform's ROI engine, providing an audit log of the sync event.
3.  **Financial Context Mapping**: Exposes the "Context Multipliers" used by the engine, allowing admins to see the underlying assumptions (e.g., "High impact multiplier for EU Finance sector").

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 12-35 | **Financial View Transformation**: Aggregates `roi_opportunities`. The logic parses the `impact_amount` as a float on line 25 to ensure the frontend can perform real-time summing and sorting on the dollar values. |
| 37-54 | **Backlog Reconciliation**: Orchestrates a `featureROIService.calculateRoadmapImpact` call. The reason for the manual audit log on line 47 is to provide a "Snapshot" of how many items were analyzed, which is critical for verifying the integrity of the financial projection over time. |
| 56-58 | **Transparency Layer**: Returns the `contextMultipliers`. This logic is responsible for explaining the "Why" behind the numbers, showing users exactly how industry-specific factors are affecting their ROI scores. |

---

## surveyAgentController (`surveyAgentController.js`)

The `surveyAgentController` is the primary interface for the platform's adaptive feedback engine, enabling intelligent survey design and conversational data collection.

### How it Works
It coordinates two specialized AI agents: the **SurveyDesignerAgent**, which translates high-level business goals (e.g., "Reduce friction in checkout") into a structured series of visual blocks, and the **Adaptive Interview Engine**, which dynamically adjusts the survey path in real-time based on the sentiment and content of the user's previous answers.

### Functionalities and Features

1.  **Generative Survey Design**: Transmutes raw goals into a full survey model including title, description, and interactive question blocks.
2.  **Conversational "Step" Logic**: Instead of a static list of questions, this engine determination the "Next Step" based on AI inference, allowing for deep-dive questions if a user expresses a specific pain point.
3.  **Multi-Dimensional Analytics**: Aggregates categorical data into four distinct insights: Sentiment Distribution, Key Drivers, Emerging Topics (Natural Language), and Response Trends.
4.  **Structural Auditing**: Allows admins to "Test" a survey structure against an AI auditor to find confusing phrasing or logic loops before launch.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 14-37 | **Strategic Provisioning**: Passes a goal to the `SurveyDesignerAgent`. The logic immediately persists the AI-generated design to the DB, ensuring that "Agentic Creativity" is instantly converted into a manageable platform asset. |
| 57-90 | **Adaptive Progression Engine**: The core of the "Conversational Survey." If no `sessionId` exists (line 64), it initializes a new state; otherwise, it passes the user's natural language answer to the `SurveyService` to infer the next most relevant question. |
| 97-134 | **Intelligence Aggregation**: Executes four parallel SQL queries to build the "Analytics Dashboard." The reasoning for splitting these into `trend`, `sentiment`, `drivers`, and `topics` is to provide both quantitative and qualitative views of the feedback loop. |

---

## surveyController (`surveyController.js`)

The `surveyController` manages the foundational lifecycle and relational storage of the platform's survey catalog.

### How it Works
This controller acts as the high-speed storage layer for the platform's feedback mechanisms. It handles the basic CRUD (Create, Read, Update, Delete) operations for the `surveys` table, ensuring that even if the AI engines are offline, the basic metadata and inventory of surveys remain accessible to the tenant.

### Functionalities and Features

1.  **Catalog Retrieval**: Provides a chronological view of all surveys active within a specific tenant context.
2.  **Atomic Survey Creation**: Allows for the rapid manual creation of survey records, establishing the "Container" for subsequent AI-driven design or manual block entry.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 5-9 | **Tenant-Isolated Inventory**: Fetches surveys filtered by the `tenantId`. The use of `asyncHandler` ensures that any database connectivity issues are caught and piped to the global error middleware without crashing the worker. |
| 20-29 | **Record Initialization**: Inserts a new survey record. The logic returns the `row` immediately (line 28), allowing the frontend to transition from a "New" modal to the "Survey Builder" interface with a valid primary key. |

---

## telemetryController (`telemetryController.js`)

The `telemetryController` provides the critical "Observability Bridge" between the user's browser and the platform's technical monitoring stack.

### How it Works
It serves as a log-ingestion sink. It accepts arrays of client-side events (errors, warnings, performance metrics), sanitizes them to prevent malicious injection, and pipes them directly into the backend's `logger.log` system with a `[FRONTEND]` tag for easy filtering in the log management tool.

### Functionalities and Features

1.  **Client-Side Error Ingestion**: Captures unhandled JavaScript exceptions or failed network requests from the end-user's device for forensic analysis.
2.  **Performance Trace Collection**: Aggregates frontend metrics (like First Contentful Paint) into the platform's global telemetry dashboard.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 4-19 | **Sanitization & Piping**: Iterates through the inbound `logs` array. The logic on line 11 (formatting the message) and line 14 (injecting the `source: 'frontend'` context) ensures that these logs are distinguishable from server-side errors during debugging by the engineering team. |

---

## voiceController (`voiceController.js`)

The `voiceController` manages the transformation of audio interactions into actionable business intelligence.

### How it Works
It centers around the **VoiceAgent**, a specialized AI squad capable of auditing transcripts. When a call is requested, the controller doesn't just return the text; it triggers an on-demand "Smart Analysis" that calculates talk-to-listen ratios and generates an executive summary, effectively acting as an automated Quality Assurance auditor.

### Functionalities and Features

1.  **Call Recording Repository**: Retrieves call metadata and transcripts stored in the `customer_events` table, identifying voice interactions by the `call_recording` event type.
2.  **On-Demand Smart Audit**: Uses the `VoiceAgent` to synthesize "Executive Insights" from raw transcripts, identifying customer sentiment and emerging topics.
3.  **Waveform Placeholder**: Provides the structure for subsequent integration with audio blob storage for visual playback in the VoC dashboard.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 6-39 | **Event-Based Discovery**: Filters the `customer_events` table for specific voice signals. The logic on line 19 maps raw sentiment scores to a binary 'Positive/Negative' label, providing the "Headline" attribute for the call list UI. |
| 41-85 | **AI-Enrichment Logic**: Fetches a single call and checks for a transcript. The reasoning for calling `VoiceAgent.analyzeCall` on line 60 is to provide a "Rich" view that includes more than just the raw text—it identifies the strategic highlights of the conversation. |

---

## webhookController (`webhookController.js`)

The `webhookController` is the high-bandwidth ingestion point for real-time updates from third-party platforms (HubSpot, Salesforce, Stripe).

### How it Works
It implements the **"Fast ACK" (Asynchronous Acknowledgment)** pattern. External providers often have strict 3-5 second timeouts. To ensure the platform never misses a signal, the controller immediately returns a `202 Accepted` status to the provider and then dispatches the complex ingestion logic to the `WebhookIngestionService` in the background.

### Functionalities and Features

1.  **Provider-Agnostic Entry**: A single endpoint that can receive webhooks from any vendor, routing the payload based on a `:provider` URL parameter.
2.  **Scalable Async Ingestion**: Decouples the receipt of the data from its processing, ensuring that even during high-traffic spikes, the platform remains responsive.
3.  **Traceable Failure Logging**: If an ingestion fails after the ACK, the original provider context and error message are logged for administrative review.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 13-16 | **The Reliability ACK**: Immediately sends a 202 status. This logic is the most critical part of the ingestion pipeline—it prevents external vendors from retrying the same event incorrectly due to server "slowness." |
| 19 | **Background Handover**: Dispatches the payload to the `WebhookIngestionService`. This service is responsible for the actual "Thinking"—identifying the customer, updating their sentiment, and triggering any relevant AI reflexes. |

---

## searchController (`searchController.js`)

The `searchController` serves as the platform's universal discovery engine, providing a unified search interface across disparate data silos.

### How it Works
It implements a "Federated Search" pattern using `Promise.all` to query three separate engines in parallel: Postgres ILIKE for relational models (Customers), Vector Embeddings for semantic patterns (Knowledge Base), and Keyword matching for system configurations (Integrations). This ensures sub-second response times regardless of data volume.

### Functionalities and Features

1.  **Direct Relational Discovery**: Performs fuzzy matching on customer names and emails, returning direct deep-links to their CRM profiles.
2.  **Semantic Context Matching**: Interfaces with the `VectorStore` to find conceptual matches in the tenant's private knowledge base, even if exact keywords aren't present.
3.  **Configuration Search**: Allows administrators to quickly locate and manage connected third-party platforms by provider name or internal label.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 14-35 | **Parallel Federation Engine**: Executes three distinct queries simultaneously. The reason for the `LIMIT 5` and `LIMIT 3` constraints is to ensure the "Quick Search" UI remains readable and prioritizes the most relevant entries from each category. |
| 38-60 | **Unified Result Normalization**: Maps raw data from SQL and Vector engines into a consistent "Title/Subtitle/Type" UI object. The fallback calculation for `doc-` IDs on line 47 ensures the frontend can handle results that might not have a primary key in the relational database. |

---

## securityController (`securityController.js`)

The `securityController` manages the platform's multi-factor authentication (MFA) lifecycle and user-level security hardening.

### How it Works
It implements a standard TOTP (Time-based One-Time Password) infrastructure. The controller coordinates with the `TwoFactorService` to generate cryptographically secure secrets and QR codes, requiring a successful "Verification Token" before the 2FA flag is persisted to the user's permanent record.

### Functionalities and Features

1.  **MFA Provisioning**: Generates unique secrets and mobile-compatible QR codes for users to scan with apps like Google Authenticator.
2.  **Secure Confirmation Workflow**: Requires a valid token before enablement, preventing lockout situations caused by incorrect secret entry.
3.  **SOC 2 Audit Compliance**: Acts as the gateway for enabling core account security features required for high-compliance enterprise tenants.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 5-19 | **Secret Provisioning Gateway**: Generates a standard TOTP secret. The logic returns both the raw `secret` and the `qrCodeUrl` to ensure the user can manually enter the code if the camera-scan fails. |
| 21-36 | **Strict Verification Logic**: Validates a one-time token against the provided secret. The reasoning for calling `enable2FA` ONLY after verification is to guarantee that the user has a functioning device before MFA is enforced at login. |

---

## settingsController (`settingsController.js`)

The `settingsController` is the administrative hub for tenant-level customization, compliance, and developer integration settings.

### How it Works
It manages five distinct functional areas, heavily utilizing PostgreSQL's **JSONB (JSON Binary)** capabilities. This allows the platform to store complex, nested configurations (like branding schemas or webhook events) with high performance and flexibility without complex relational schema migrations.

### Functionalities and Features

1.  **White-Label Branding Engine**: Manages the tenant's visual identity (colors, fonts), ensuring a consistent experience across the platform's UI.
2.  **Web-Hook Orchestration**: Handles the CRUD lifecycle and secure secret management for outbound notifications to external systems.
3.  **API Hardening**: Generates and revokes live-prefixed API keys (`sk_live_`), storing only their secure hashes to maintain Zero-Trust security standards.
4.  **GDPR/CCPA Compliance Hub**: Provides atomic "Request Purge" and "Export Data" endpoints to fulfill legal data requests for customer identity protection.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 32-50 | **Setting Injection Logic**: Updates the `settings` JSONB blob. The reasoning for the read-modify-write pattern on line 39 is to ensure that updating "Branding" does not accidentally overwrite other unrelated tenant-level flags. |
| 136-157 | **Zero-Knowledge Key Generation**: Produces a live API key and its hash. The logic returns the `rawSecret` exactly once to the user (line 152) and only stores the `secretHash`, ensuring that even a database breach cannot leak usable API keys. |
| 176-200 | **Compliance Delegation**: Interfaces with the `complianceService`. This abstraction ensures that the complex legal logic of finding and deleting user data across multiple tables (purging) is handled outside the HTTP controller layer. |
| 218-227 | **Deep JSONB Update**: Uses the `jsonb_set` function. This PostgreSQL-native logic allows for an atomic update of a specific path (e.g., `'{domains}'`) within a larger configuration document, preventing race conditions during simultaneous setting updates. |

---

## ssoController (`ssoController.js`)

The `ssoController` handles enterprise-grade authentication via Single Sign-On (SSO) protocols.

### How it Works
It implements the **JIT (Just-In-Time) Provisioning** pattern. When a user successfully authenticates via SAML or OIDC, the controller automatically creates a platform user record with safe default permissions (Viewer) if they don't already exist. This eliminates the need for manual user onboarding for large organizations.

### Functionalities and Features

1.  **Auto-Provisioning Entry Point**: Transitions corporate identity profiles into authenticated platform sessions with 24-hour JWT issuance.
2.  **Audit-Ready Logging**: Records every success and failure with IP and User-Agent tracking, providing the "Paper Trail" required for security audits.
3.  **Metadata Service**: Serves the Service Provider (SP) XML descriptors required for enterprise admins to configure the platform as a trusted app in Okta/Azure AD.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 23-41 | **JIT Identity Upsert**: Checks for existing users before provisioning. The reasoning for the `'Viewer'` default role on line 36 is a "Security-First" approach, requiring an admin to manually promote the user after their initial auto-onboarding. |
| 50-61 | **Forensic Login Audit**: Dispatches a log entry to the `AuditService`. By capturing the `RelayState` and IP address, this logic helps detect "Identity Spoofing" or suspicious cross-region login attempts. |
| 75-87 | **SP Descriptor Engine**: Returns the SAML metadata XML. This logic is strictly for administrative onboarding, enabling a "Plug-and-Play" setup with standard identity providers. |

---

## strategyController (`strategyController.js`)

The `strategyController` manages the platform's high-level business maps and the "Next Best Action" (NBA) intelligence engine.

### How it Works
It manages "Strategy Decks"—visual directed graphs representing business objectives. It coordinates with a specialized `strategyAgent` that "Audits" these decks in the background, correlating live metrics to the nodes in the graph to find strategic gaps or "At-Risk" initiatives.

### Functionalities and Features

1.  **Strategy Graph Management**: CRUD for business maps, visualizing the relationship between "Goals" and "Metrics."
2.  **Agentic Strategic Audit**: Triggers a deep AI scan of the current strategy to identify misalignments and generate prioritized "Next Best Actions."
3.  **NBA Management Hub**: Serves the prioritized recommendations produced by the AI agents, allowing admins to approve or dismiss tactical moves.
4.  **Metric Health Monitoring**: Automatically updates the health status of strategy nodes based on real-time data ingestion.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 103-142 | **Agentic Monitoring Loop**: Passes the strategy graph to the `strategyAgent`. The reasoning for the `hasChanges` check on line 122 is to prevent unnecessary database writes and visual flicker in the UI if the AI finds the strategy currently balanced. |
| 190-220 | **Recommendation Prioritization**: Fetches non-dismissed NBAs. The logic includes a specific error guard (line 214) to gracefully handle environments where the NBA table hasn't been migrated yet, returning an empty set instead of a platform error. |
| 225-269 | **Atomic NBA Upsert**: Saves AI recommendations using a conflict-handling pattern. If a similar recommendation already exists, the logic updates its `priority` and `updated_at` (line 239) instead of creating a duplicate, keeping the Action Center clean. |

---

## creativeController (`campaigns/creativeController.js`)

The `creativeController` is responsible for the generative aspect of marketing campaigns, specifically focusing on the creation and refinement of creative content for various platforms.

### How it Works
It leverages the `contentGenService` to interface with LLMs for generating marketing copy variations and refining existing content. It acts as a specialized assistant for campaign creators, providing structured creative outputs based on topics, audiences, and goals.

### Functionalities and Features

1.  **Generative Copy Creation**: Transmutes high-level campaign parameters (topic, audience, goal) into multiple creative variations optimized for specific platforms (e.g., Slack, Email, Social).
2.  **Interactive Content Refinement**: Provides a "Conversational Editor" interface where existing copy can be refined through natural language instructions (e.g., "Make it more professional", "Add a call to action").

### Feature Logic Descriptions

-   **Creative Synthesis**: The `generateCreative` logic takes a payload of marketing context and dispatches it to the `contentGenService`, which uses a RAG-enhanced prompt to ensure the output matches the brand's voice and the target audience's psychographics.
-   **Instruction-Driven Editing**: Instead of rewrite-and-replace, the `refineCreative` logic applies specific transformations to existing text, allowing for iterative improvement of campaign assets.

### Line-by-Line Explanation
| Line | Logic & Implementation Explanation |
| :--- | :--- |
| 1-2 | **Creative Stack Dependencies**: Imports the `contentGenService` for AI-driven text generation and the `campaignService` for potential context-aware campaign linking. These services abstract the complexity of LLM prompting and campaign management. |
| 4-12 | **Targeted Variation Engine**: The `generateCreative` method extracts campaign parameters from the request body. The reasoning behind the `await contentGenService.generateVariations` call is to provide the user with a diverse set of options simultaneously, reducing the time from "Concept" to "Draft." |
| 14-22 | **Iterative Refinement Path**: Implements the logic for polishing creative assets. By passing specific `instructions` to the generator, this endpoint enables a "Human-in-the-loop" workflow where the AI acts as a tireless copyeditor for the final campaign assets. |
