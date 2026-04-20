# Heimdall MCP Tool Reference

23 tools across four domains. All tools run on Heimdall's hosted infrastructure at `https://api.heimdallgov.com/mcp/sse` and require a Bearer API key with a tier of `core`, `api`, `pro`, or `enterprise`.

**Read tools are safe** — they query production data without mutation. **Write tools** (2 total) are explicitly marked and gated to your subscription's own state only.

---

## Findings & Research (11 tools)

### `search_findings`
Search the unified findings database (GAO, IG, FAC). Filter by keyword (matches `finding_title`), agency, severity (high/moderate/low), status (open/closed), or minimum questioned costs. Returns finding metadata with source-document traceability.

### `get_open_recommendations`
Get overdue GAO/IG recommendations ordered by urgency. Filter by agency or urgency tier (Critical/High/Elevated). Returns recommendation text, age in days, priority status, and linked finding info.

### `search_source_documents`
Search the permanent source document archive (OIG reports, GAO, budget docs). All findings trace back to source documents stored with full PDF retention. Filter by `source_type` (oig/gao/budget), agency, date range, or keyword.

### `get_oig_publications`
Browse OIG publications scraped from 70+ Inspector General websites. Filter by agency, `doc_type` (report/testimony/press/work_plan), or `report_type` (audit/inspection/evaluation/investigation).

### `search_gao_recommendations`
Search GAO recommendations (raw data from oversight.gov / GAO.gov). Filter by keyword, agency, priority status, or implementation status. Priority recommendations are flagged by GAO as most impactful.

### `extract_entities`
Extract named entities from a report's source PDF with page-level citations. Entities include federal agencies, audit terms (from GAGAS Yellow Book), CFR/USC citations, finding IDs, and monetary amounts.

### `find_similar_findings`
Find the N most similar findings to a target finding using TF-IDF cosine similarity. Each result includes the similar finding's metadata and source PDF path for cross-reference.

### `get_domain_keywords`
Get TF-IDF distinctive terms for an agency or the full corpus. If an agency is specified, returns terms distinctive for that agency compared to the corpus average (high distinctiveness = domain-specific language).

### `analyze_topics`
Analyze topic distribution across the findings corpus using LDA. Returns topics with their top words and representative findings. If an agency is specified, shows which topics that agency's findings cluster into.

### `get_cross_agency_patterns`
Find systemic issues that appear across multiple agencies. Uses KMeans clustering on TF-IDF vectors to group similar findings. Returns clusters spanning at least `min_agencies` different agencies — useful for identifying government-wide problems.

### `search_documents`
Semantic + full-text search across the Heimdall RAG corpus. Hybrid vector + full-text search over **197K+ PDF chunks** (GAO/IG reports) and **66K+ structured data chunks** (federal register, regulations). Returns ranked passages with source citations.

---

## Sentinel — live oversight monitoring (6 tools)

Sentinel scrapes 70+ OIG sites, GAO, SAM.gov, and budget sources nightly. It detects changes and anomalies so your agent can surface what's new without re-scanning the full corpus.

### `get_sentinel_status`
Get the latest Sentinel run status per source group. Returns the most recent run for each `source_group` (oversight, budget, gao, sam) with status, counts, and timing. Filter by `source_group` to narrow.

### `list_recent_changes`
List recent changes detected by Sentinel scrapers. Filter by `source_group`, `change_type` (new/modified/validation_failure), `severity` (info/warning/critical), or time window (days).

### `query_alerts`
Query Sentinel anomaly alerts. Filter by `source_group`, `severity` (info/warning/critical), and acknowledgement status. Defaults to unacknowledged alerts only.

### `acknowledge_alert` ⚠️ **write operation**
Mark a Sentinel alert as acknowledged. Returns confirmation with the acknowledgement timestamp. Scoped to your own subscription's alerts.

### `get_oig_profile_health`
Get health status of OIG scraper profiles. Shows each OIG's last scrape date, publication count, and error status. Filter by `oig_id` for a single agency.

### `get_baseline_comparison`
Compare recent Sentinel runs against historical baselines. Computes average counts and detects anomalies (values > 2 std devs from the mean). Use this to identify unusual scraper behavior.

---

## Pipeline & Market Intel (6 tools)

These tools surface outputs of the ranked-opportunity pipeline, including NAICS spending trends, cost tracking, and AI usage telemetry.

### `list_pipeline_runs`
List recent pipeline runs with filtering. Filter by `profile_id` (e.g., `mgo-cpa`) or `status` (success/failed/running). Returns runs ordered by date with key metrics.

### `get_cost_trends`
Get cost-per-item trend over recent pipeline runs. Returns `run_date`, `ranked_count`, `estimated_cost_usd`, `cost_per_item`, token counts, and `run_mode` for the last N successful runs. Filter by `profile_id`.

### `get_latest_ranked`
Get the latest ranked opportunities from the most recent successful run. Filter by `profile_id`, minimum score, or signal type (Active RFP, Likely Recompete, Emerging Trend, Intel Only).

### `get_spending_trends`
Get NAICS spending trends from the latest pipeline run. Filter by `naics_code` or signal type (growing/declining/stable). Shows year-over-year spending changes for federal market analysis.

### `approve_rule_proposal` ⚠️ **write operation**
Approve a pending rule proposal for the next pipeline run. Changes the proposal status from `pending` to `approved`. The rule will be applied on the next run.

### `get_ai_usage`
Query AI API usage logs with filtering. Filter by `module` (scoring/extraction/triage/profiler), model name, or time window. Returns token counts, costs, and quality metrics.

---

## Authorization model

Every tool call passes through `MCPAuthMiddleware`, which:
1. Validates the Bearer token against the `api_keys` table (SHA-256 hash match, `is_active = true`).
2. Checks the tier is in `{core, api, pro, enterprise, admin}`.
3. Records `last_used_at` for audit.
4. Applies per-tier rate limits on the shared REST rate-limiter.

A failed check returns HTTP 401/403 before the SSE handshake completes, so misconfigured clients fail fast.

## Rate limits

Per tier, enforced at the API gateway (not the MCP gateway specifically — shared with REST):

| Tier | Requests/min |
|---|---|
| core | 60 |
| api | 300 |
| pro | 600 |
| enterprise | negotiated |

Rate-limit headers (`X-RateLimit-Remaining`, `X-RateLimit-Reset`) are returned on responses. Exceeding returns HTTP 429.

## Feedback

Tool naming, parameters, and descriptions will evolve. Open an issue at [github.com/getgovernmentmoney/heimdall-mcp/issues](https://github.com/getgovernmentmoney/heimdall-mcp/issues) with suggestions, bug reports, or requests for new tools.
