# Heimdall MCP Tool Reference

**36 tools across nine domains.** All tools run on Heimdall's hosted infrastructure at `https://api.heimdallgov.com/mcp/sse` and require a Bearer API key with a tier of `core`, `api`, `pro`, or `enterprise`.

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

---

## Contracts (4 tools)

The FPDS + USASpending-backed contract search surface — 58.5M+ contract actions across all agencies and NAICS codes. Matches BGOV, GovWin, and Fed-Spend contract-history features at 1/100th the price.

### `search_contracts`
General-purpose contract search. Filter by contracting/funding agency substring, NAICS code (exact), vendor name or UEI, description keyword, signed-since date, value range (min/max), or set-aside type (`8A`, `WOSB`, `HUBZone`, `SDVOSBC`). Each result includes PIID, vendor, agency, NAICS, POP end date, contract value ceiling, competition, and a direct FPDS link.

### `find_expiring_contracts`
Find contracts coming up for recompete — contracts with POP ending in the next N months (default 18). Filter by agency or NAICS. Results sorted soonest-expiring first. Each result includes `days_until_expiry` for sorting and prioritization. This is the recompete-tracking feature that Fed-Spend and GovWin charge 12–50x more for.

### `get_vendor_contracts`
Get all contracts for a vendor by UEI (preferred, exact match) or name substring. Returns the latest modification of each PIID the vendor has held. Use to assess competitor win history, prospect portfolio, or teaming partner experience.

### `get_contract_detail`
Get full detail for a single contract by PIID. Returns the contract header plus, if `include_modifications=True`, every modification row ordered by signed date. Use to trace cost growth, schedule slippage, and option exercise history.

---

## Contractor Risk & Performance (2 tools)

The Heimdall differentiators — no competitor ($9 to $50K/yr) entity-resolves OIG/GAO findings and FAC outcomes into contractor risk profiles. 872,793 entities scored.

### `get_contractor_risk_profile`
Get the **6-dimension contractor risk profile** for a vendor. Provide `uei` (preferred, exact) or `legal_name` (substring). Returns composite score, risk tier (Low/Moderate/Elevated/High/Critical), top risk driver, and each sub-score:

| Dimension | Weight | Source |
|---|---|---|
| Oversight exposure | 26% | OIG/GAO direct findings |
| Compliance/integrity | 21% | SAM exclusions + FAC |
| Award concentration | 17% | single-agency dependency, sole-source rate |
| Performance | 15% | FPDS modifications (cost growth, schedule) |
| Financial signals | 13% | USASpending revenue trend, award gaps |
| Environment risk | 8% | declining NAICS, regulatory pressure |

### `get_contractor_performance`
Get the **5-subdimension performance score** for a vendor. Scored 0-100 from public FPDS modification data — labeled as "Performance Indicators (Public Data)" for honesty (these are proxy signals, not CPARS):

- `cost_control_score` — modification cost growth ratio
- `schedule_score` — POP extensions
- `satisfaction_score` — option exercise rate
- `integrity_score` — FAPIIS events, exclusions, sustained protests
- `continuity_score` — contract extensions, recompete wins

---

## Grants (3 tools)

Federal grants and Single Audit compliance surface.

### `search_grant_awards`
Search historical federal grant awards (USAspending). Filter by recipient name/UEI, CFDA/Assistance Listing, awarding agency, place-of-performance state, or minimum amount. Returns award ID, FAIN, recipient info, agency, CFDA title, start/end dates, award amount, and USAspending URL.

### `search_grant_opportunities` _(data ingestion pending — returns status message)_
Open grant opportunities from Grants.gov. Full ingestion lands in Sprint 8c (~2026-04-27). Current response: `status: data_ingestion_pending` with a pointer to `search_grant_awards` for historical data.

### `get_fac_findings` _(data ingestion pending — returns status message)_
Single Audit (FAC) findings for grant recipients. Full ingestion lands in Sprint 8c. In the interim, use `search_findings` for OIG and GAO findings which are already loaded (10,145 findings across 5,000+ reports).

---

## Legislation (2 tools)

Federal bill and resolution search via the Congress API.

### `search_legislation`
Search Congressional bills and resolutions. Filter by keyword (title/description/latest action), congress number (e.g., 119), chamber (`house`/`senate`), or bill type (`hr`, `s`, `hjres`, `sjres`, `hconres`, `sconres`, `hres`, `sres`). Returns bills ordered by most recent update.

### `get_bill_detail`
Get detail for a specific bill by number (e.g., `HR 1234`, `S 500`). If multiple Congresses have the same bill number, pass `congress_num` to disambiguate.

---

## Appropriations (1 tool)

Federal agency budget data from OMB — parity with BGOV's agency-budget feature.

### `get_agency_budget`
Get agency/bureau/account-level budget authority with 3-year trend (requested vs prior-year enacted vs two-year-back actual). Filter by agency name, bureau name, account keyword, fiscal year, or minimum budget authority. Returns `delta_pct` showing requested vs prior-enacted change — use to spot programs with budget pressure or growth.

---

## Additional Findings Tool

### `get_findings_by_contractor`
**The Heimdall differentiator** — returns OIG/GAO/FAC findings filed against a specific contractor, by UEI (exact) or name (substring). Filter by status (open/closed). No competitor links oversight findings to contractors at the entity level. Pair with `get_contractor_risk_profile` for the full 6-dim composite picture.

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
