---
name: ocas-thread
description: Personal web activity interpretation. Reconstructs browsing sessions and research threads from searches, site visits, engagement signals, and downloads. Derives interests, source affinities, and seen/unseen status. Emits journals and Chronicle candidates for downstream skills. Private skill -- do not distribute.
metadata: {"openclaw":{"emoji":"🧵"}}
---

# Thread

Thread interprets browser activity to reconstruct coherent lines of exploration over time. It ingests browsing activity, organizes it into sessions and research threads, derives engagement and interest signals, and emits journals and Chronicle candidates for downstream skills.

Thread is observational only. It does not browse the web, replay sessions, write raw logs to Chronicle, execute actions, or make recommendations directly.

This skill is private and must never be published or distributed.

## When to use

Thread operates in the background. It is useful when the system needs to know:
- What the user has already seen
- What topics the user is actively researching
- Which sources are useful or unhelpful
- What interests are strengthening over time
- Which searches ended in satisfaction vs frustration
- What content is new vs already consumed

## When not to use

- Web research — use Sift
- Knowledge graph writes — use Elephas
- Pattern discovery — use Corvus
- Preference persistence — use Taste
- Action execution — use Praxis

## Responsibility boundary

Thread owns browsing-derived behavioral interpretation: ingestion, normalization, session reconstruction, research thread reconstruction, source affinity, seen/unseen checks, interest candidate detection, and signal emission.

Thread does not own: durable memory (Elephas/Chronicle), preferences (Taste), pattern discovery (Corvus), web search (Sift), task actions (Praxis).

## Core principle

Raw browsing history is not memory. Raw events remain in Thread's local datastore. Only derived signals, structured summaries, and promotion candidates may be proposed to Elephas. This prevents Chronicle from becoming a browser log archive.

## Inputs

### Mode 1: Local Near-Realtime Ingestion
Chrome History SQLite database (read from a copied snapshot, not the live locked file).
Expected source: `~/Library/Application Support/Google/Chrome/Default/History`
Polling target: every 60-120 seconds.

### Mode 2: Remote Delayed Sync
A local exporter pushes normalized browser activity. Suggested cadence: daily. Must produce the same normalized event schema.

## Normalized event schema

All ingestion paths normalize to a shared event model with fields: event_id, event_type, timestamp, url, domain, title, search_query, referrer_url, referrer_domain, visit_duration, engagement_signal, device, source, session_hint, metadata.

Supported event types: search_query, web_visit, typed_navigation, download, bookmark_add, bookmark_open, form_submission, redirect.

## Derived signals

**Dwell time** — computed from timestamps and navigation order.

**Engagement signal** — classified per visit: bounce, short_click (0-10s), neutral_click (10-60s), long_click (>60s), deep_engagement (>180s). Upper-bound noise guard for implausible idle windows.

**Query reformulation** — detect evolving intent sequences (e.g., "surya" → "surya ocr" → "surya ocr benchmarks").

**Source affinity** — repeated long-clicks, revisits, and typed navigation strengthen domain affinity.

**Seen/unseen** — tracks whether a URL, domain, or resource has been seen. Useful to Sift and Praxis.

**Research depth** — long navigation chains, revisits, and downloads increase thread depth score.

## Session reconstruction

Group events into browsing sessions using: time proximity, referrer chains, search-result click chains, overlapping entities, continuous topical behavior. Default heuristic: new session if inactivity > 20-30 minutes. Merge events that clearly belong together even if timer would split them.

Session output: session_id, start_time, end_time, primary_queries, domains_visited, long_click_count, downloads, candidate_entities, candidate_concepts, engagement_summary.

## Research thread reconstruction

A research thread is a coherent line of exploration spanning multiple sessions across days or weeks. Merge sessions using: overlapping search terms, overlapping entities, overlapping concepts, repeated domains, temporal recurrence, reformulation continuity.

Thread output: thread_id, topic_label, first_seen, last_seen, sessions, primary_queries, entities, concepts, sources, engagement_summary, thread_strength, novelty_score, interest_candidate.

## Interest detection

Flag interest_candidate = true when: sessions >= 3, long_clicks >= 3, entity/concept overlap is high, behavior spans more than one day or session cluster.

Thread does not create Taste records directly. Interest candidates are passed to Corvus and Elephas through journals and intake files.

## Commands

- `thread.recent_searches` — recent search queries
- `thread.recent_visits` — recent site visits
- `thread.search_history` — search history for a query or topic
- `thread.has_seen` — check if a URL or domain has been visited
- `thread.active_threads` — currently active research threads
- `thread.recent_threads` — most recent research threads
- `thread.thread_for_topic` — research thread for a specific topic
- `thread.preferred_sources` — preferred sources for a topic
- `thread.interest_candidates` — current interest candidates
- `thread.research_summary` — summary of research on a topic
- `thread.domain_affinity` — affinity score for a domain
- `thread.long_click_sources` — sources with long-click engagement for a topic
- `thread.status` — ingestion state, session count, thread count, last activity
- `thread.journal` — write journal for the current run; called at end of every run

## Inter-skill interfaces

Thread writes research thread signals to Corvus at: `~/openclaw/data/ocas-corvus/intake/{thread_id}.json`
Written when a research thread reaches sufficient depth to be an interest candidate.

Thread writes Chronicle candidates to Elephas at: `~/openclaw/db/ocas-elephas/intake/{candidate_id}.signal.json`
Written only for high-quality candidates: interest candidates with session_count >= 3, long_click_count >= 3. Raw browsing events must never be written to Chronicle.

Good candidates: stable research topics, durable interests, preferred sources, repeated entity/topic relationships.
Bad candidates: raw URL visits, short clicks, single-page bounces.

See `spec-ocas-interfaces.md` for signal format and handoff contracts.

## Privacy and data minimization

- Ignore chrome:// pages, new-tab pages, extension pages
- Ignore localhost unless explicitly configured
- Deduplicate repeated instant reloads
- Cap or discard implausible idle dwell windows
- Prefer derived insight over raw retention

## Storage layout

```
~/openclaw/data/ocas-thread/
  config.json
  events.jsonl
  sessions.jsonl
  threads.jsonl
  interests.jsonl
  decisions.jsonl

~/openclaw/journals/ocas-thread/
  YYYY-MM-DD/
    {run_id}.json
```

The OCAS_ROOT environment variable overrides `~/openclaw` if set.

Default config.json:
```json
{
  "skill_id": "ocas-thread",
  "skill_version": "2.0.0",
  "config_version": "1",
  "created_at": "",
  "updated_at": "",
  "ingestion": {
    "mode": "local",
    "poll_interval_seconds": 90,
    "chrome_profile": "Default"
  },
  "engagement": {
    "short_click_max_seconds": 10,
    "neutral_click_max_seconds": 60,
    "long_click_min_seconds": 60,
    "deep_engagement_min_seconds": 180,
    "idle_cap_seconds": 3600
  },
  "session": {
    "inactivity_gap_minutes": 25
  },
  "interest": {
    "min_sessions": 3,
    "min_long_clicks": 3
  },
  "retention": {
    "days": 90,
    "max_records": 50000
  }
}
```

## OKRs

Universal OKRs from spec-ocas-journal.md apply to all runs.

```yaml
skill_okrs:
  - name: session_reconstruction_accuracy
    metric: fraction of sessions correctly grouping related events
    direction: maximize
    target: 0.95
    evaluation_window: 30_runs
  - name: thread_merge_precision
    metric: fraction of thread merges confirmed correct
    direction: maximize
    target: 0.90
    evaluation_window: 30_runs
  - name: chronicle_candidate_precision
    metric: fraction of Chronicle candidates confirmed useful by Elephas
    direction: maximize
    target: 0.90
    evaluation_window: 30_runs
  - name: raw_event_chronicle_leak
    metric: count of raw browsing events written to Chronicle
    direction: minimize
    target: 0
    evaluation_window: 30_runs
```

## Optional skill cooperation

- Elephas — receives Chronicle candidate signals via intake directory
- Corvus — receives research thread signals via intake directory
- Sift — may use Thread context for query rewriting (cooperative, not dependency)
- Taste — may use repeated stable thread signals to inform preferences
- Praxis — may use seen/unseen checks to avoid redundant suggestions
- Mentor — may analyze Thread journals to improve thresholds and merge logic

Thread must function when all cooperating skills are absent.

## Journal outputs

Observation Journal — all ingestion, session reconstruction, and thread analysis runs.

Thread emits three journal subclasses (all Observation type):
- thread_activity_journal — low-level normalized activity events
- thread_session_journal — session reconstructions
- thread_research_journal — research thread reconstructions (primary cross-skill artifact)

## Visibility

private

## Support file map

File | When to read
`references/schemas.md` | Before creating events, sessions, threads, or candidates
`references/journal.md` | Before thread.journal; at end of every run
