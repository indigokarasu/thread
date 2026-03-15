---
name: ocas-thread
description: Reconstructs Chrome browser sessions and research threads to derive engagement signals and interest candidates. Proposes stable topics and sources to Elephas as Chronicle candidates. Use when asked to analyze browsing history, reconstruct research activities, check seen/unseen status, detect emerging interests, or provide thread context for downstream skills (Corvus, Sift, Elephas, Praxis, Taste). Trigger for any request involving browser activity interpretation, session reconstruction, or research thread analysis.
metadata: {"openclaw":{"emoji":"🧵","requires":{"bins":["python3","sqlite3"]}}}
---

# ocas-thread

Ingest Chrome browser activity, reconstruct sessions and research threads, derive engagement and interest signals, and emit structured journals with Chronicle promotion candidates.

Thread observes only. It does not browse, replay sessions, write to Chronicle directly, or execute actions.

---

## Core Responsibility

Given a snapshot of Chrome History, thread:

1. **Normalizes** browser events to canonical schema (search queries, web visits, typed navigation, downloads, bookmarks, form submissions, redirects)
2. **Classifies** engagement: dwell time, interaction depth, signal strength
3. **Reconstructs sessions** by merging adjacent events with topical continuity
4. **Reconstructs research threads** by merging overlapping sessions (shared entities, concepts, domains, query reformulation)
5. **Detects interest candidates** by scoring stability, recurrence, and engagement depth
6. **Proposes Chronicle candidates** to Elephas for durable memory storage
7. **Exposes tools** for seen/unseen checks, thread context, preferred sources, and research summaries

## Ingestion

### Mode 1: Local Near-Realtime (Mac with active Chrome)

Run `scripts/snapshot_chrome_history.py` to safely copy the locked History database before querying.

Default source path:
```
~/Library/Application Support/Google/Chrome/Default/History
```

Recommended polling: every 60–120 seconds.

### Mode 2: Remote Delayed Sync

Exporter on user machine normalizes events and pushes on schedule (suggested: daily). Remote sync must emit the same normalized event schema as local ingestion.

**Critical:** Always snapshot before querying. Never read from the live locked database directly.

---

## Normalization

Convert all ingested events to the canonical schema before any downstream processing.

**Schema reference:** Read `references/event_schema.md` for field definitions and rules.

**Supported event types:** `search_query`, `web_visit`, `typed_navigation`, `download`, `bookmark_add`, `bookmark_open`, `form_submission`, `redirect`

**Compute for every event:**
- **Dwell time:** `next_navigation_timestamp - current_timestamp`
- **Engagement signal:** bounce / short_click / neutral_click / long_click / deep_engagement (see `references/session_reconstruction.md` for thresholds)
- **Query reformulation:** Detect escalating specificity chains (e.g., `llm` → `llm inference` → `llm inference optimization`)
- **Source affinity:** Track domain revisits, long-clicks, and typed navigation (see `references/session_reconstruction.md`)

---

## Session Reconstruction

Merge normalized events into sessions using:
- Time proximity (inactivity gap: 20–30 min, configurable)
- Referrer chains
- Search result click chains
- Overlapping entities and concepts
- Continuous topical behavior

**Principle:** Prefer merging events that belong together over strict timer-based splits.

**Full merge logic:** Read `references/session_reconstruction.md`

---

## Research Thread Reconstruction

Merge sessions into research threads using:
- Overlapping search terms
- Overlapping extracted entities
- Overlapping concepts
- Repeated domains and source affinity
- Temporal recurrence
- Query reformulation continuity

**Merge rule:** Any condition sufficient:
- ≥2 shared entities (high confidence)
- 1 high-confidence shared entity + strong query similarity
- Clear reformulation chain continuity

**Full merge logic and thread schema:** Read `references/session_reconstruction.md`

---

## Ontology

Thread uses Chronicle-aligned ontology from `references/thread_ontology.md`.

**Node classes:** Entity, Concept, Thing, Activity, Thread, Source, Interest, Signal

**Relationship types:** researched, visited, searched_for, engaged_with, downloaded, reformulated_to, led_to, part_of_thread, indicates_interest_in, prefers_source_for, has_seen

---

## Output Journals

Emit three journal types. **Full field definitions:** Read `references/journal_spec.md`

| Journal | Purpose | Audience |
|---------|---------|----------|
| `thread_activity_journal` | Low-level normalized events; dwell analysis and debugging | Debug / downstream analysis |
| `thread_session_journal` | Session bursts with topical focus; immediate intent grouping | Real-time activity tracking |
| `thread_research_journal` | Cross-session research threads with stable topics and candidates | **Primary output:** feeds Corvus, Elephas, Taste, Sift, Mentor |

**Default primary artifact:** `thread_research_journal`

---

## Chronicle Promotion

Thread never writes to Chronicle directly. Propose candidates to Elephas via `research_journal.chronicle_candidates` field.

**Good candidates:** Stable topics, durable interests, preferred sources, recurring entity relationships

**Bad candidates:** Individual URL visits, single-click events, short-duration visits

**Promotion rules and examples:** Read `references/chronicle_boundary.md`

---

## Interest Detection

Flag `interest_candidate: true` when all hold:
- ≥3 sessions on the topic
- ≥3 long_click events
- High entity or concept overlap
- Behavior spans ≥2 days or session clusters

Pass flagged interests to Corvus and Elephas via research journal. Thread does not create Taste records directly.

---

## Tool Surface

```
thread.recent_searches(n)              # Last n search queries
thread.recent_visits(n)                # Last n web visits
thread.search_history(query)           # All searches matching query
thread.has_seen(url_or_domain)         # Seen/unseen check
thread.active_threads()                # Currently open research threads
thread.recent_threads(n)               # Last n completed threads
thread.thread_for_topic(topic)         # Thread matching topic
thread.preferred_sources(topic)        # Highest-affinity domains
thread.interest_candidates()           # Detected interests with scores
thread.research_summary(topic)         # Summary of research on topic
thread.domain_affinity(domain)         # Engagement score for domain
thread.long_click_sources(topic)       # Sources visited deeply
```

All functions return structured results from derived signals, never raw database rows.

---

## Privacy and Data Minimization

**Ignored:** `chrome://` pages, new-tab pages, extension pages, localhost (unless configured), instant reload duplicates.

**Applied:** Idle dwell noise guard (see `references/session_reconstruction.md`)

**Data retention:** Raw events remain local. Only derived signals and promotion candidates leave thread.

---

## Skill Cooperation

Thread functions independently but cooperates with other skills when present:

| Skill | Thread provides |
|-------|-----------------|
| Elephas | Research journals + Chronicle candidates |
| Corvus | Research journals for pattern and novelty detection |
| Sift | Recent thread context for query rewriting |
| Praxis | Seen/unseen status for deduplication |
| Taste | Stable thread signals for preference proposal |
| Mentor | Journals for threshold and heuristic refinement |

---

## Scripts

| Script | Purpose |
|--------|---------|
| `scripts/snapshot_chrome_history.py` | Safe snapshot of locked Chrome History DB before ingestion |
| `scripts/validate_thread_output.py` | Validates output journal schema compliance |

---

## Reference Files

| File | When to read |
|------|-------------|
| `references/event_schema.md` | Before normalizing ingested events |
| `references/session_reconstruction.md` | Before computing dwell signals, sessions, threads, or affinity |
| `references/thread_ontology.md` | When extracting entities or building graph candidates |
| `references/journal_spec.md` | When emitting any journal type |
| `references/chronicle_boundary.md` | When deciding what to propose as Chronicle candidates |
