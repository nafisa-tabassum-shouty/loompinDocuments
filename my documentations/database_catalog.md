# Loompin VoC: Complete Database Catalog (v2.0)

This document provides a 100% exhaustive inventory of the Loompin Voice of Customer (VoC) platform database structure, grouped by functional domain.

---

## 1. Core Platform & Infrastructure
The foundation of the platform, managing multi-tenancy, security, and baseline operations.

### `tenants`
Enterprise registry; stores plan levels, status, and regional deployment config.
| id | name | plan | status | region |
|---|---|---|---|---|
| `550e8400-e29b-41d4-a716-446655440000` | Acme Corp | Enterprise | active | gcp-us-central1 |
| `671f9511-f30c-4874-a690-349023450000` | Global Tech | Pro | active | aws-eu-west-1 |
| `982f2345-d41c-4321-b112-998877660000` | Stark Ind | Free | active | azure-us-east |
| `123e4567-e89b-12d3-a456-426614174000` | Wayne Ent | Enterprise | active | gcp-us-central1 |
| `a1b2c3d4-e5f6-g7h8-i9j0-k1l2m3n4o5p6` | Cyberdyne | Trial | suspended | aws-ap-northeast-1 |

### `users`
Platform user accounts linked to tenants.
| id | tenant_id | email | name | role |
|---|---|---|---|---|
| `782a2222-b333-4444-a555-666677778888` | `550e8400-e29b-41d4-a716-446655440000` | admin@acme.com | Sarah Chen | Admin |
| `903b3333-c444-5555-b666-777788889999` | `550e8400-e29b-41d4-a716-446655440000` | support@acme.com | Mike Rossi | User |
| `112c4444-d555-6666-c777-888899990000` | `671f9511-f30c-4874-a690-349023450000` | dev@global.tech | Jane Doe | Developer |
| `223d5555-e666-7777-d888-999900001111` | `982f2345-d41c-4321-b112-998877660000` | tony@stark.com | Tony Stark | Owner |
| `334e6666-f777-8888-e999-000011112222` | `123e4567-e89b-12d3-a456-426614174000` | bruce@wayne.com | Bruce Wayne | Admin |

### `user_security`
MFA tokens, login attempts, and account lockout state.
| user_id | two_factor_enabled | failed_attempts | lockout_until | last_login_ip |
|---|---|---|---|---|
| `782a2222-b333-...` | true | 0 | NULL | 192.168.1.1 |
| `903b3333-c444-...` | false | 2 | NULL | 10.0.0.5 |
| `112c4444-d555-...` | true | 0 | NULL | 172.16.0.10 |
| `223d5555-e666-...` | true | 1 | NULL | 8.8.8.8 |
| `334e6666-f777-...` | false | 0 | NULL | 1.1.1.1 |

### `api_keys`
Secure tokens for external service authentication.
| id | tenant_id | name | prefix | status |
|---|---|---|---|---|
| `key_001` | `550e8400-e29b-...` | Dashboard Bot | `lp_live_...` | Active |
| `key_002" | `671f9511-f30c-...` | CI/CD Worker | `lp_test_...` | Active |
| `key_003" | `982f2345-d41c-...` | Zapier Hook | `lp_live_...` | Revoked |
| `key_004" | `123e4567-e89b-...` | Mobile App | `lp_prod_...` | Active |
| `key_005" | `a1b2c3d4-e5f6-...` | External CRM | `lp_dev_...` | Expired |

### `integrations`
Connectors for Salesforce, Zendesk, Slack, etc.
| id | tenant_id | provider | category | status |
|---|---|---|---|---|
| `int_001` | `550e8400-e29b-...` | Salesforce | CRM | Connected |
| `int_002` | `550e8400-e29b-...` | Zendesk | Support | Connected |
| `int_003` | `671f9511-f30c-...` | Slack | Messaging | Syncing |
| `int_004" | `982f2345-d41c-...` | HubSpot | Marketing | Failed |
| `int_005" | `123e4567-e89b-...` | Intercom | Support | Connected |

### `sync_history`
Audit trail of data ingestion from external integrations.
| id | integration_id | status | records_synced | duration_ms |
|---|---|---|---|---|
| `sync_901` | `int_001` | success | 1250 | 4500 |
| `sync_902" | `int_002` | success | 450 | 2100 |
| `sync_903" | `int_003` | running | NULL | NULL |
| `sync_904" | `int_004` | error | 0 | 120 |
| `sync_905" | `int_005` | success | 890 | 3200 |

### `audit_logs`
Partitioned ledger of all system and user actions (Strategic Audit).
| id | tenant_id | user_id | action | entity_type |
|---|---|---|---|---|
| `aud_101` | `550e8400-e29b-...` | `782a2222...` | USER_LOGIN | ACCOUNT |
| `aud_102` | `550e8400-e29b-...` | `782a2222...` | KEY_CREATED | API_KEYS |
| `aud_103` | `671f9511-f30c-...` | `112c4444...` | SYNC_START | INTEGRATIONS |
| `aud_104" | `982f2345-d41c-...` | `223d5555...` | PLAN_UPGRADE | TENANTS |
| `aud_105" | `123e4567-e89b-...` | `334e6666...` | POLICY_UPDATE | GOVERNANCE |

### `system_settings`
Global and tenant-specific configuration flags.
| key | value | type | description |
|---|---|---|---|
| `max_api_rps` | `100` | global | Rate limiting threshold |
| `ai_model_default` | `gpt-4o` | global | Default LLM for analysis |
| `retention_days" | `365` | tenant | Data retention period |
| `enable_mfa_force" | `true` | tenant | Enforce MFA for all users |
| `theme_color" | `#0f172a` | tenant | UI Primary Color |

### `webhooks`
Outbound event notification configurations.
| id | tenant_id | url | event_types | status |
|---|---|---|---|---|
| `wh_001` | `550e8400-e29b-...` | `https://webhook.site/1` | `["event.new"]` | active |
| `wh_002` | `671f9511-f30c-...` | `https://api.acme.com/v1` | `["sync.fail"]` | inactive |
| `wh_003` | `982f2345-d41c-...` | `https://hooks.slack.com/...` | `["alert.high"]` | active |
| `wh_004" | `123e4567-e89b-...` | `https://requestbin.io/x` | `["user.add"]` | active |
| `wh_005" | `a1b2c3d4-e5f6-...` | `https://my-app.com/hook` | `["*"]` | active |

### `platform_team`
Managed list of Loompin internal operators.
| id | email | access_level | status |
|---|---|---|---|
| `plt_001` | ops@loompin.com | SuperAdmin | active |
| `plt_002" | support_lead@loompin.com | Admin | active |
| `plt_003" | dev_ops@loompin.com | Write | active |
| `plt_004" | analyst@loompin.com | Read | active |
| `plt_005" | trainee@loompin.com | ViewOnly | onboarding |

### `plans`
Billing tier definitions and feature access maps.
| name | price_monthly | max_users | features |
|---|---|---|---|
| Free | 0.00 | 5 | `["basic_dash"]` |
| Pro | 49.00 | 25 | `["ai_analytics", "integrations"]` |
| Enterprise | 499.00 | 999 | `["sso", "dedicated_infra", "custom_nlp"]` |
| Startup | 19.00 | 10 | `["basic_ai"]` |
| Trial | 0.00 | 2 | `["all_features"]` |

### `validation_rules`
Data integrity checks for the ingestion pipeline.
| entity_type | field_name | rule_type | params |
|---|---|---|---|
| `customer` | `email` | `regex` | `^.+@.+\..+$` |
| `event` | `occurred_at` | `not_null` | `{}` |
| `event` | `sentiment_score` | `range` | `{"min": -1, "max": 1}` |
| `product` | `sku` | `unique` | `{"scope": "tenant"}` |
| `order` | `amount` | `positive` | `{}` |

---

## 2. Customer Intelligence & VoC Data
Storage for raw feedback signals and derived customer profiles.

### `customers`
Unified profiles of end-users whose feedback is being analyzed.
| id | tenant_id | email | name | health_score |
|---|---|---|---|---|
| `cust_001` | `550e8400-e29b-...` | j.doe@gmail.com | John Doe | 85 |
| `cust_002` | `550e8400-e29b-...` | a.smith@yahoo.com | Alice Smith | 42 |
| `cust_003` | `671f9511-f30c-...` | b.wayne@wayne.com | Bruce Wayne | 99 |
| `cust_004" | `982f2345-d41c-...` | s.stark@stark.com | Star Lord | 15 |
| `cust_005" | `123e4567-e89b-...` | p.parker@daily.com | Peter Parker | 78 |

### `customer_events`
High-velocity signal stream (surveys, tickets, call logs), partitioned for performance.
| id | tenant_id | event_type | sentiment_score | summary |
|---|---|---|---|---|
| `evt_101` | `550e8400-e29b-...` | support_ticket | -0.45 | "App crashes on startup" |
| `evt_102` | `550e8400-e29b-...` | survey_response | 0.90 | "Love the new dark mode!" |
| `evt_103` | `671f9511-f30c-...` | intercom_chat | 0.12 | "How do I upgrade my plan?" |
| `evt_104" | `982f2345-d41c-...` | app_store_review | -0.95 | "Worst update ever. 1 star." |
| `evt_105" | `123e4567-e89b-...` | tweet_mention | 0.55 | "@Loompin is saving me hours!" |

### `events`
General system events and legacy feedback signals.
| id | tenant_id | event_name | severity | metadata |
|---|---|---|---|---|
| `sys_001` | `global` | NODE_UP | info | `{"node": "worker-1"}` |
| `sys_002` | `550e8400-e29b-...` | SYNC_COMPLETE | success | `{"source": "SF"}` |
| `sys_003` | `671f9511-f30c-...` | ALERT_THRESHOLD | critical | `{"metric": "latency"}` |
| `sys_004" | `982f2345-d41c-...` | LOGIN_FAIL | warning | `{"user": "tony@..."}` |
| `sys_005" | `123e4567-e89b-...` | REFLEX_EXEC | info | `{"rule": "auto-tag"}` |

### `surveys` & `survey_topics`
Definitions and categorization for VoC survey tools.
| id | tenant_id | title | status |
|---|---|---|---|
| `sur_001` | `550e8400-e29b-...` | Q1 NPS | active |
| `sur_002" | `671f9511-f30c-...` | Retention Survey | draft |
| `sur_003" | `982f2345-d41c-...` | Trial Exit | active |
| `sur_004" | `123e4567-e89b-...` | Product Satisfaction | paused |
| `sur_005" | `a1b2c3d4-e5f6-...` | Mobile Experience | active |

### `survey_drivers`
Extracted KPIs that influence survey sentiment.
| topic | impact_score | trend | category |
|---|---|---|---|
| Shipping Speed | -0.15 | worsening | Logistics |
| App Performance | 0.25 | improving | Product |
| Billing Issue | -0.12 | stable | Finance |
| Agent Quality | 0.08 | improving | Support |
| Feature Request | 0.05 | stable | Enhancement |

### `survey_response_trends`
Time-series snapshots of feedback volume.
| survey_id | date | total_responses | avg_sentiment |
|---|---|---|---|
| `sur_001` | 2026-02-01 | 150 | 0.45 |
| `sur_001" | 2026-02-02 | 185 | 0.42 |
| `sur_002" | 2026-02-01 | 45 | 0.12 |
| `sur_002" | 2026-02-02 | 52 | 0.15 |
| `sur_003" | 2026-02-01 | 12 | -0.10 |

### `survey_sentiment`
NLP-derived sentiment scores per survey response.
| response_id | overall_score | joy | anger | fear |
|---|---|---|---|---|
| `res_001` | 0.85 | 0.92 | 0.01 | 0.02 |
| `res_002" | -0.45 | 0.05 | 0.65 | 0.12 |
| `res_003" | 0.12 | 0.35 | 0.15 | 0.05 |
| `res_004" | -0.92 | 0.01 | 0.95 | 0.45 |
| `res_005" | 0.55 | 0.65 | 0.10 | 0.05 |

### `nps_events`
Specific ledger for Net Promoter Score collection.
| id | customer_id | score | status | channel |
|---|---|---|---|---|
| `nps_001` | `cust_001` | 10 | submitted | email |
| `nps_002" | `cust_002` | 4 | submitted | in-app |
| `nps_003" | `cust_003` | 8 | pending | sms |
| `nps_004" | `cust_004` | 0 | submitted | help-desk |
| `nps_005" | `cust_005` | 9 | submitted | web |

### `win_loss_analytics`
Analysis of sales outcomes and competitive feedback.
| deal_id | outcome | competitor | reason_category | sentiment |
|---|---|---|---|---|
| `opp_101` | WON | Oracle | AI Insight | positive |
| `opp_102" | LOST | Salesforce | Pricing | negative |
| `opp_103" | WON | None | Automation | positive |
| `opp_104" | LOST | Zendesk | Existing Ecosystem | neutral |
| `opp_105" | WON | HubSpot | Ease of Implementation | positive |

### `identity_graph`
Links multiple customer IDs across different platforms to a single person.
| canonical_id | source_platform | source_id | confidence |
|---|---|---|---|
| `cust_001` | Salesforce | `sf_7788` | 0.99 |
| `cust_001" | Zendesk | `zen_4455` | 0.95 |
| `cust_002" | Shopify | `shp_1122` | 0.85 |
| `cust_003" | Intercom | `int_9900` | 0.98 |
| `cust_005" | Twitter | `tw_8822` | 0.75 |

### `taxonomy_topics`
Master list of industry-specific feedback categories.
| id | label | parent_id | weight |
|---|---|---|---|
| `tax_001` | Logistics | NULL | 1.0 |
| `tax_002" | Shipping Speed | `tax_001` | 0.8 |
| `tax_003" | Product Quality | NULL | 1.0 |
| `tax_004" | Support Politeness | NULL | 0.6 |
| `tax_005" | AI Accuracy | NULL | 1.2 |

### `topic_aliases` & `topic_assignments`
Mappings between raw feedback text and the central taxonomy.
| topic_id | entity_id | score | assigned_at |
|---|---|---|---|
| `tax_002` | `evt_101` | 0.95 | 2026-02-01 |
| `tax_003" | `evt_102` | 0.82 | 2026-02-01 |
| `tax_001" | `evt_103` | 0.75 | 2026-02-02 |
| `tax_005" | `evt_104` | 0.98 | 2026-02-02 |
| `tax_004" | `evt_105` | 0.65 | 2026-02-02 |

---

## 3. Analytics & Strategic Planning
Tables used for deep-dive research, simulations, and project tracking.

### `roi_opportunities`
AI-predicted business improvements with attached financial metrics.
| id | title | type | impact_amount | status |
|---|---|---|---|---|
| `roi_101` | SSO Login Fix | Bug | 900000 | active |
| `roi_102" | Checkout Opt | Feature | 650000 | proposed |
| `roi_103" | Support Bot | Feature | 450000 | approved |
| `roi_104" | Churn Guard | Initiative | 1200000 | planning |
| `roi_105" | API Health | Bug | 150000 | completed |

### `experience_drivers` & `driver_history`
Root-cause analysis for CSAT/NPS shifts.
| id | topic | impact_score | sentiment | category |
|---|---|---|---|---|
| `drv_001` | Shipping Delay | -0.42 | negative | Ops |
| `drv_002" | Support Empathy | 0.22 | positive | Service |
| `drv_003" | Feature Gap | -0.31 | negative | Product |
| `drv_004" | UI Speed | 0.15 | positive | Tech |
| `drv_005" | Pricing Model | -0.18 | negative | Billing |

### `journey_flows`
Sankey data representing customer transitions.
| id | name | total_volume | status |
|---|---|---|---|
| `jrn_f_001` | Support Resolution | 12500 | active |
| `jrn_f_002" | Onboarding Funnel | 8900 | active |
| `jrn_f_003" | Upgrade Path | 4200 | active |
| `jrn_f_004" | Win-Back Campaign | 1500 | draft |
| `jrn_f_005" | Cancellation Exit | 600 | active |

### `journeys`
Logical definitions of customer lifecycle paths.
| id | name | steps_count | success_rate |
|---|---|---|---|
| `jrn_001` | Enterprise Trial | 12 | 0.45 |
| `jrn_002" | Free User Self-Serve | 5 | 0.82 |
| `jrn_003" | Support Case Escalation | 8 | 0.95 |
| `jrn_004" | Global Store Visit | 4 | 0.12 |
| `jrn_005" | Partner API Onboard | 15 | 0.35 |

### `segments` & `segment_perception`
Customer cohort definitions and their emotional profiles.
| id | name | size | primary_trait |
|---|---|---|---|
| `seg_001` | High-Value Saas | 1250 | Loyal |
| `seg_002" | EMEA Retailers | 8900 | Price-Sensitive |
| `seg_003" | US Enterprise | 450 | Feature-Hungry |
| `seg_004" | APAC SMB | 12000 | Speed-Focused |
| `seg_005" | Churn Risks | 300 | Frustrated |

### `segment_versions`
Audit trail of cohort logic changes.
| segment_id | version | changed_by | change_reason |
|---|---|---|---|
| `seg_001` | 1 | admin | initial creation |
| `seg_001" | 2 | scientist_1 | updated revenue threshold |
| `seg_002" | 1 | admin | regional grouping |
| `seg_003" | 1 | system | auto-detected cohort |
| `seg_005" | 1 | support_lead | manual definition |

### `projects`
High-level strategic initiatives (Initiatives Hub).
| id | title | status | owner | progress |
|---|---|---|---|---|
| `prj_001` | Churn Reduction | On Track | Sarah C. | 45% |
| `prj_002" | Support Automation | At Risk | Mike R. | 15% |
| `prj_003" | Market Expansion | On Track | Tony S. | 80% |
| `prj_004" | Infrastructure v2 | Delayed | Jane D. | 5% |
| `prj_005" | Client Alpha | Completed | Bruce W. | 100% |

### `project_forecasts`
Impact projections for planned initiatives.
| project_id | metric | low_est | high_est | confidence |
|---|---|---|---|---|
| `prj_001` | Retention % | 5.2 | 8.5 | 0.85 |
| `prj_001" | Savings $ | 450000 | 800000 | 0.70 |
| `prj_002" | Hours Saved | 1200 | 1800 | 0.90 |
| `prj_003" | New ARR | 1200000 | 3000000 | 0.60 |
| `prj_004" | Latency ms | -150 | -250 | 0.95 |

### `roadmap_items`
Public or internal feature roadmaps derived from feedback.
| id | title | status | eta | priority |
|---|---|---|---|---|
| `rdm_101` | Dark Mode | Done | 2026-01 | High |
| `rdm_102" | API v3 | In Progress | 2026-03 | Critical |
| `rdm_103" | Mobile Offline | Backlog | 2026-06 | Medium |
| `rdm_104" | Audit Pro | Planning | 2026-05 | Low |
| `rdm_105" | Global Search | In Progress | 2026-02 | High |

### `strategy_maps`
Collaborative canvases for planning business goals.
| id | title | node_count | view_mode | status |
|---|---|---|---|---|
| `str_001` | 2026 Vision | 25 | Interactive | Published |
| `str_002" | Ops Expansion | 12 | Static | Draft |
| `str_003" | Tech Modernize | 45 | Graph | Historical |
| `str_004" | HR Growth | 5 | List | Archive |
| `str_005" | Customer First | 18 | Interactive | Draft |

### `ab_tests` & `ab_events`
Experimentation framework for testing platform variations.
| id | name | variant_a_pct | status | winner |
|---|---|---|---|---|
| `abt_001` | Button Color | 50 | Completed | Blue |
| `abt_002" | Landing Copy | 50 | In-Progress | NULL |
| `abt_003" | Model v2 | 40 | Active | NULL |
| `abt_004" | Feed Layout | 50 | Completed | Grid |
| `abt_005" | Pricing Tier | 50 | Failed | NULL |

### `simulation_runs`
Logs from AI agents running "what-if" scenarios.
| id | scenario_name | run_time_ms | outcome | confidence |
|---|---|---|---|---|
| `sim_001` | Black Friday Load | 45000 | Stable | 0.99 |
| `sim_002" | DB Failure Path | 12000 | Recovered | 0.85 |
| `sim_003" | Global Churn Spk | 8500 | High Risk | 0.70 |
| `sim_004" | New Feature Adp | 3200 | Success | 0.92 |
| `sim_005" | Regional Outage | 1500 | Panic | 0.50 |

---

## 4. Product Ecosystem (Master Catalog)
Enterprise-grade product management with tiered pricing and intelligence.

### `products`
Central product registry with rich JSONB attributes.
| id | name | brand | status | version |
|---|---|---|---|---|
| `p_001` | X-Phone 15 | Pear | active | v1 |
| `p_002" | UltraBook Pro | Zen | active | v3 |
| `p_003" | SmartWatch Active | Fit | draft | v1 |
| `p_004" | Solar Charger v2 | Sunny | archived | v2 |
| `p_005" | VR Goggles Nano | Vision | active | v1 |

### `product_variants`
Specific SKUs (size, color, version) for products.
| id | product_id | sku | stock_level | differentiator |
|---|---|---|---|---|
| `var_001` | `p_001` | P-15-B-128 | 1250 | Black / 128GB |
| `var_002" | `p_001` | P-15-W-256 | 850 | White / 256GB |
| `var_003" | `p_002` | Z-UB-S-512 | 420 | Silver / 512GB |
| `var_004" | `p_003` | F-SW-B-SM | 1500 | Black / Small |
| `var_005" | `p_005` | V-GG-N-STD | 200 | Standard |

### `product_intelligence`
Agent-detected issues and optimization scores for products.
| product_id | quality_score | sentiment_score | issue_count |
|---|---|---|---|
| `p_001` | 95 | 0.88 | 1 |
| `p_002" | 78 | 0.45 | 5 |
| `p_003" | 15 | -0.10 | 12 |
| `p_004" | 42 | 0.12 | 0 |
| `p_005" | 89 | 0.92 | 2 |

### `product_versions`
Historical versions of product specs.
| product_id | version | changelog | created_at |
|---|---|---|---|
| `p_001` | v1.0 | Initial release | 2025-01-01 |
| `p_001" | v1.1 | Fixed camera spec | 2025-03-01 |
| `p_002" | v3.0 | M3 Processor | 2025-10-15 |
| `p_004" | v2.0 | Added Solar Panel | 2025-06-01 |
| `p_005" | v1.0 | Beta Launch | 2026-01-20 |

### `product_channels`
Mappings of where products are sold (Web, Retail, App).
| product_id | channel | is_available | priority |
|---|---|---|---|
| `p_001` | Web | true | 1 |
| `p_001" | Retail | true | 2 |
| `p_001" | App | true | 3 |
| `p_002" | Web | true | 1 |
| `p_003" | Excl_Beta | true | 1 |

### `product_connections`
Cross-sell and up-sell logic (Related Products).
| product_a_id | product_b_id | relation_type | score |
|---|---|---|---|
| `p_001` | `p_005` | Up-sell | 0.85 |
| `p_001" | `p_003` | Cross-sell | 0.92 |
| `p_002" | `p_001` | Accessory | 0.78 |
| `p_005" | `p_001` | Required | 1.00 |
| `p_001" | `p_004` | Bundle | 0.65 |

### `product_scenes`
Visual configuration data for 3D product previews.
| product_id | scene_url | environment | model_format |
|---|---|---|---|
| `p_001` | `/3d/p1.glb` | Studio | GLB |
| `p_002" | `/3d/p2.obj` | Office | OBJ |
| `p_005" | `/3d/v1.usdz` | Cyber | USDZ |
| `p_003" | `/3d/s1.gltf` | Park | GLTF |
| `p_004" | `/3d/sc1.glb` | Rooftop | GLB |

### `price_books` & `price_entries`
Tiered, multi-currency, and VIP pricing engine.
| name | currency | is_active | priority |
|---|---|---|---|
| Standard USD | USD | true | 0 |
| VIP Euro | EUR | true | 10 |
| Summer Sale | USD | false | 100 |
| Partner JPY | JPY | true | 50 |
| Employee Free | USD | true | 999 |

### `features`
Granular feature list for software or hardware products.
| product_id | name | type | status |
|---|---|---|---|
| `p_001` | 5G Sub-6 | Core | Active |
| `p_001" | Night Mode | Software | Released |
| `p_002" | 120Hz Liquid | Hardware | Active |
| `p_005" | Eye Tracking | Beta | Testing |
| `p_003" | Heart Sync | Core | Draft |

---

## 5. Autonomous Operations & Agentic Swarm
Logging and configuration for the AI agents that power the "Neural Core".

### `reflexes`
Automated trigger-action rules (The "Pulse").
| id | name | type | trigger_condition | action_plan |
|---|---|---|---|---|
| `rfx_001` | Auto-Tag Negative | Event | `sentiment < -0.5` | `tag_entity('urgent')` |
| `rfx_002" | Support Alert | Threshold | `ticket_spike > 2x` | `notify_slack('support')` |
| `rfx_003" | Churn Warning | Prediction | `health < 30` | `assign_csm()` |
| `rfx_004" | Lead Route | CRM | `new_deal == true` | `route_to_ae()` |
| `rfx_005" | Refund Check | Policy | `refund_req == true` | `validate_policy()` |

### `reflex_logs`
Audit trail of reflexive actions executed by the system.
| reflex_id | entity_id | outcome | execution_time_ms |
|---|---|---|---|
| `rfx_001` | `evt_101` | success | 450 |
| `rfx_002" | `global` | alerted | 120 |
| `rfx_003" | `cust_002` | success | 890 |
| `rfx_001" | `evt_104` | success | 320 |
| `rfx_005" | `deal_44` | rejected | 1500 |

### `action_logs` & `unified_action_logs`
Comprehensive history of agent-driven tasks.
| id | agent_id | action_type | status | duration |
|---|---|---|---|---|
| `act_001` | `swarm_1` | SCRAPE_REDDIT | success | 12.5s |
| `act_002" | `swarm_2` | SUMMARIZE_VOC | success | 5.2s |
| `act_003" | `swarm_1` | ANALYZE_SENTIMENT | running | NULL |
| `act_004" | `swarm_3` | GENERATE_STRATEGY | failed | 45.0s |
| `act_005" | `swarm_1` | SYNC_CRM | success | 8.1s |

### `next_best_actions`
AI-suggested steps for support or sales teams.
| entity_id | suggestion | confidence | impact |
|---|---|---|---|
| `cust_002` | Send Discount Code | 0.92 | high |
| `opp_104" | Schedule Demo | 0.85 | medium |
| `ticket_44" | Escalate to Eng | 0.98 | critical |
| `cust_005" | Share Beta Inv | 0.70 | low |
| `opp_102" | Update Pricing | 0.65 | medium |

### `playbooks`
Standardized procedures for agents to follow during incidents.
| id | title | steps_json | category | status |
|---|---|---|---|---|
| `ply_001` | Server Outage | `["detect", "reroute", "alert"]` | Infra | ready |
| `ply_002" | PR Crisis | `["monitor", "draft_resp", "post"]` | Social | draft |
| `ply_003" | Data Breach | `["lockdown", "audit", "notify"]` | Security | ready |
| `ply_004" | Support Surge | `["prioritize", "auto_reply"]` | Support | active |
| `ply_005" | Competitor Mv | `["analyze", "shift_str"]` | Market | ready |

### `wargame_messages` & `wargame_simulations`
Multi-agent collaborative planning sessions.
| simulation_id | agent_sender | message_context | round_index |
|---|---|---|---|
| `war_001` | `analyst_1` | "Detecting pricing shift" | 1 |
| `war_001" | `strategist_1` | "Recommend discount response" | 2 |
| `war_001" | `legal_1` | "Policy allows 10% max" | 3 |
| `war_002" | `ops_1` | "Infra load spike detected" | 1 |
| `war_002" | `ops_2` | "Rerouting to node-B" | 2 |

### `outcome_ledger`
The Final ROI Verification ledger; tracks actual business impact.
| revenue_impact | operational_hrs_saved | verification_status | user_id |
|---|---|---|---|
| 5000.00 | 12.5 | Verified | `user_1` |
| 12000.00 | 45.0 | Pending | NULL |
| 0.00 | 5.2 | Disputed | `user_2` |
| 850.00 | 1.5 | Verified | `user_1` |
| 250000.00 | 120.0 | Verified | `admin` |

### `governance_policies`
Compliance rules that agents must adhere to.
| id | name | type | logic | status |
|---|---|---|---|---|
| `pol_001` | No PII Scraping | Safety | `redact(email, phone)` | active |
| `pol_002" | Budget Limit | Cost | `max_spend($50/day)` | active |
| `pol_003" | Human in Loop | Approval | `req_appr(refund > $100)` | active |
| `pol_004" | GDPR Compliance | Legal | `delete_req(30_days)` | active |
| `pol_005" | Model Safety | Ethics | `block_offensive()` | active |

### `permission_policies` & `security_policies`
RBAC and security constraints for AI operations.
| entity | action | role | effect |
|---|---|---|---|
| `DATABASE` | `DROP` | `PLATFORM_ADMIN` | Allow |
| `USER_DATA` | `READ` | `ANALYST` | Allow_Masked |
| `BILLING` | `WRITE` | `FINANCE` | Allow |
| `SECURITY` | `UPDATE` | `USER` | Deny |
| `SWARM_EXEC` | `RESTART` | `OPS` | Allow |

---

## 6. Supply Chain & Enterprise Logistics
Operational data for physical products and enterprise security.

### `suppliers`
Registry of vendor and manufacturing partners.
| id | name | category | rating | status |
|---|---|---|---|---|
| `sup_001` | TechFlow Inc | Electronics | 4.8 | active |
| `sup_002" | Global Logistics | Shipping | 4.2 | active |
| `sup_003" | Fabrikon | Manufacturing | 3.5 | probation |
| `sup_004" | SecureSys | Security | 5.0 | active |
| `sup_005" | EcoPack | Packaging | 4.5 | active |

### `supply_nodes`
Geographical distribution centers and factory locations.
| id | supplier_id | location | node_type | capacity_pct |
|---|---|---|---|---|
| `node_001` | `sup_001` | Shenzhen, CN | Factory | 85 |
| `node_002" | `sup_002` | Hamburg, DE | Warehouse | 42 |
| `node_003" | `sup_002` | Long Beach, US | Port | 99 |
| `node_004" | `sup_003` | Mumbai, IN | Hub | 15 |
| `node_005" | `sup_005` | Austin, US | Factory | 60 |

### `supply_orders`
Tracking of raw material or product movement.
| id | node_id | sku | quantity | status |
|---|---|---|---|---|
| `ord_901` | `node_001` | P-15-B-128 | 5000 | Shipped |
| `ord_902" | `node_002" | Z-UB-S-512 | 1200 | Processing |
| `ord_903" | `node_003" | F-SW-B-SM | 10000 | Arrived |
| `ord_904" | `node_004" | V-GG-N-STD | 200 | Delayed |
| `ord_905" | `node_005" | S-CHARG-V2 | 850 | Draft |

### `cost_drivers`
FinOps definitions (AI token costs, storage rates).
| driver_key | category | unit_cost | unit_label |
|---|---|---|---|
| `ai.input.1k` | AI | 0.000500 | 1k tokens |
| `ai.output.1k` | AI | 0.001500 | 1k tokens |
| `infra.storage.gb` | Infra | 0.023000 | GB |
| `telecom.sms.seg` | Telecom | 0.007500 | segment |
| `infra.compute.sec` | Infra | 0.000040 | sec |

### `tenant_usage_daily`
Daily resource consumption ledger for billing.
| tenant_id | usage_date | driver_key | quantity | cost |
|---|---|---|---|---|
| `acme_1` | 2026-02-01 | `ai.input.1k` | 125000 | 62.50 |
| `acme_1" | 2026-02-01 | `infra.storage.gb` | 850 | 19.55 |
| `global_tech` | 2026-02-01 | `ai.output.1k` | 45000 | 67.50 |
| `stark_ind` | 2026-02-01 | `telecom.sms.seg` | 1200 | 9.00 |
| `wayne_ent` | 2026-02-01 | `infra.compute.sec` | 1000000 | 40.00 |

### `invoices`
Billing history and PDF receipts for tenants.
| id | tenant_id | date | amount | status |
|---|---|---|---|---|
| `INV-24-001` | `acme_1` | 2026-01-01 | 499.00 | Paid |
| `INV-24-002" | `acme_1" | 2026-02-01 | 545.20 | Pending |
| `INV-24-003" | `global_tech` | 2026-02-01 | 1280.00 | Paid |
| `INV-24-004" | `stark_ind` | 2026-02-01 | 19.00 | Overdue |
| `INV-24-005" | `wayne_ent` | 2026-02-01 | 499.00 | Paid |

### `sso_providers`
SAML/OIDC configuration for enterprise tenants (Okta, Azure).
| tenant_id | provider_type | entry_point | enabled |
|---|---|---|---|
| `acme_1` | Okta | `https://okta.acme.com/...` | true |
| `global_tech" | Azure | `https://login.microsoft.com/...` | true |
| `wayne_ent" | Google | `https://accounts.google.com/...` | true |
| `stark_ind" | Custom | `https://sso.stark.com/...` | false |
| `lex_corp" | Auth0 | `https://lex.auth0.com/...` | true |

---

## 7. Performance Monitoring & Dashboards
Pre-aggregated data and real-time health metrics.

### `database_metrics`
QPS, Latency, and Error rates for database nodes.
| integration_id | avg_latency | p95_latency | throughput | error_rate |
|---|---|---|---|---|
| `db_prod_1` | 24ms | 120ms | 4200 | 0.0001 |
| `db_prod_2` | 15ms | 85ms | 1200 | 0.0000 |
| `db_replica_1` | 8ms | 25ms | 8500 | 0.0000 |
| `db_test_1` | 120ms | 500ms | 50 | 0.0500 |
| `db_analytics` | 450ms | 1.2s | 200 | 0.0010 |

### `database_query_heatmap` & `database_slow_queries`
Diagnostic data for bottlenecks.
| hour | latency | volume | query_type | impact |
|---|---|---|---|---|
| 09:00 | 1200ms | 15000 | Slow_JOIN | Critical |
| 12:00 | 850ms | 45000 | Normal | Medium |
| 15:00 | 2500ms | 8000 | Full_Scan | High |
| 18:00 | 120ms | 25000 | Normal | Low |
| 21:00 | 45ms | 12000 | Index_Hit | Low |

### `messaging_metrics` & `messaging_hourly_volume`
Bot/Human chat performance.
| hour | bot_handled | human_handled | handoffs | deflection_rate |
|---|---|---|---|---|
| 08:00 | 120 | 15 | 5 | 88% |
| 12:00 | 450 | 85 | 32 | 75% |
| 16:00 | 320 | 120 | 45 | 65% |
| 20:00 | 150 | 45 | 12 | 82% |
| 00:00 | 45 | 2 | 0 | 98% |

### `support_metrics` & `support_agent_performance`
Ticket volume and CSAT by agent.
| agent_name | csat | volume | avg_handle_time | status |
|---|---|---|---|---|
| Sarah | 4.8 | 150 | 12m | online |
| Mike | 4.2 | 120 | 15m | away |
| Jessica | 4.9 | 180 | 10m | online |
| Tom | 3.5 | 90 | 25m | offline |
| Alex | 4.0 | 110 | 18m | online |

### `appstore_metrics`, `appstore_ratings`, `appstore_versions`
Mobile store signals.
| platform | avg_rating | downloads | crash_free_rate | pending_reviews |
|---|---|---|---|---|
| iOS | 4.8 | 12500 | 99.9% | 120 |
| Android | 4.2 | 89000 | 98.5% | 450 |
| AppGallery | 3.5 | 1200 | 95.0% | 12 |
| GalaxyStore | 4.0 | 5000 | 97.2% | 85 |
| Windows | 4.5 | 250 | 100% | 0 |

### `competitor_metrics`, `competitor_social_feed`, `competitor_web_changes`
External market intelligence.
| competitor | metric_name | value | delta | source |
|---|---|---|---|---|
| Salesforce | Market Share | 45% | +1.2% | Gartner |
| Zendesk | NPS | 35 | -2 | Social_Audit |
| Oracle | Pricing_Index | 1.8 | +0.5 | Web_Scrape |
| HubSpot | Feature_Count | 125 | +12 | Release_Notes |
| Intercom | Ad_Spend | $1.2M | -15% | SEMRush |

### `telemetry_events`
Frontend usage logs and app performance data.
| event_id | page_url | load_time_ms | user_agent | browser |
|---|---|---|---|---|
| `tel_001` | `/dashboard` | 450 | `Chrome/120...` | Chrome |
| `tel_002` | `/feed` | 1200 | `Safari/17...` | Safari |
| `tel_003` | `/settings` | 120 | `Firefox/122...` | Firefox |
| `tel_004" | `/analytics` | 2500 | `Chrome/120...` | Chrome |
| `tel_005" | `/login` | 85 | `Edge/121...` | Edge |

### `executive_config`
Configuration for the "Executive Summary" high-level view.
| category | title | score | trend | narrative |
|---|---|---|---|---|
| Infrastructure | Strategic Intel | 88 | up | System health nominal |
| Revenue | Financial Health | 72 | down | Pipeline slowed 15% |
| Support | Service Ops | 94 | up | CSAT record high |
| Marketing | Campaign Perf | 85 | stable | ROAS 8.5x |
| Product | Commerce Pulse | 91 | up | Cart abandonment improved |

---

## 8. Knowledge Management & Hierarchy
Unstructured data and organizational structures.

### `documents`
Knowledge base articles and uploaded assets.
| id | title | type | status | size |
|---|---|---|---|---|
| `doc_001` | Onboarding Guide | PDF | published | 1.2MB |
| `doc_002" | Q2 Strategy | DOCX | draft | 450KB |
| `doc_003" | Logo Pack | ZIP | published | 15MB |
| `doc_004" | Schema Map | PNG | published | 2.5MB |
| `doc_005" | Policy Manual | PDF | archived | 800KB |

### `knowledge_embeddings`
Vector representations of documents for RAG.
| document_id | chunk_index | model_version | dim_count |
|---|---|---|---|
| `doc_001` | 0 | `text-embedding-3` | 1536 |
| `doc_001` | 1 | `text-embedding-3` | 1536 |
| `doc_002` | 0 | `text-embedding-3` | 1536 |
| `doc_005" | 0 | `text-embedding-3` | 1536 |
| `doc_004" | 0 | `text-embedding-3` | 1536 |

### `omni_files`
Multi-format file storage registry.
| id | bucket_path | file_ext | content_type | provider |
|---|---|---|---|---|
| `file_101` | `s3://l-prod/1.pdf` | .pdf | application/pdf | AWS |
| `file_102" | `gs://l-prod/logo.png` | .png | image/png | GCP |
| `file_103" | `az://l-prod/data.csv` | .csv | text/csv | AZURE |
| `file_104" | `s3://l-prod/v.mp4` | .mp4 | video/mp4 | AWS |
| `file_105" | `gs://l-prod/bk.zip` | .zip | application/zip | GCP |

### `mobile_drafts`
Synchronized state for mobile app users.
| user_id | device_id | draft_key | content_json | updated_at |
|---|---|---|---|---|
| `u_1` | `iphone_15` | `review_101` | `{"text": "Love it"}` | 2026-02-01 |
| `u_2` | `pixel_8` | `reply_44` | `{"text": "Thanks"}` | 2026-02-02 |
| `u_1` | `ipad_pro` | `review_101` | `{"text": "Love it!"}` | 2026-02-02 |
| `u_3" | `s24_ultra` | `post_1` | `{"title": "Epic"}` | 2026-02-02 |
| `u_4" | `moto_g` | `comment_9` | `{"body": "Hello"}` | 2026-02-02 |

### `reports`
Generated PDFs and analytics exports.
| id | report_type | format | generated_by | url |
|---|---|---|---|---|
| `rep_001` | Monthly_Audit | PDF | system | `/s3/r1.pdf` |
| `rep_002" | NPS_Snapshot | CSV | `user_1` | `/gs/r2.csv` |
| `rep_003" | Competitor_Analysis | PDF | `admin` | `/s3/r3.pdf` |
| `rep_004" | Yearly_Growth | XLSX | `system` | `/az/r4.xlsx` |
| `rep_005" | Error_Log | JSON | `ops_1` | `/s3/r5.json` |

### `twin_history`
State evolution logs for "Digital Twins" of customers or products.
| twin_id | state_snapshot | event_trigger | confidence |
|---|---|---|---|
| `tw_cust_1` | `{"health": 45}` | CHURN_ALRT | 0.85 |
| `tw_prod_1` | `{"stock": 0}` | OUT_OF_STK | 0.99 |
| `tw_cust_1` | `{"health": 85}` | RECOVERED | 0.92 |
| `tw_node_1` | `{"load": 95}` | TRAFFIC_SPK | 0.95 |
| `tw_cust_2" | `{"loyalty": 10}` | NEW_USER | 1.00 |

### `hierarchy_nodes` & `org_units`
Business organizational structure mapping.
| id | name | level | parent_id |
|---|---|---|---|
| `org_001` | Global HQ | 0 | NULL |
| `org_002" | North America | 1 | `org_001` |
| `org_003" | EMEA | 1 | `org_001` |
| `org_004" | Sales NA | 2 | `org_002` |
| `org_005" | Support EMEA | 2 | `org_003` |

### `user_org_map`
Mapping of users to their specific business units.
| user_id | org_id | primary_access |
|---|---|---|
| `u_1` | `org_004` | true |
| `u_2` | `org_005` | true |
| `admin` | `org_001` | true |
| `u_3" | `org_004` | false |
| `u_4" | `org_005` | true |

---

> [!IMPORTANT]
> This catalog is current as of the **`Sentient Supply`** migration phase. It covers all 85+ tables currently defined in the backend schema.
