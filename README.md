# ocas-thread

**Reconstructs Chrome browser sessions and research threads. Derives engagement signals and interest candidates. Proposes stable topics and sources to Elephas for durable memory.**

`ocas-thread` ingests Chrome History, normalizes browsing events, reconstructs user sessions and research threads, classifies engagement depth, detects emerging interests, and emits structured activity and research journals for downstream skills (Corvus, Elephas, Taste, Sift, Praxis).

Thread observes only. It does not browse, replay sessions, write to Chronicle directly, or execute actions.

---

## Overview

### What Thread Does

- Safely snapshots locked Chrome History database
- Normalizes browser events to canonical schema (searches, visits, navigation, downloads, bookmarks)
- Classifies dwell time and engagement signals
- Reconstructs sessions from topically-connected event sequences
- Reconstructs research threads by merging overlapping sessions
- Detects and ranks interest candidates by stability and recurrence
- Proposes durable topics and sources to Elephas as Chronicle candidates
- Exposes tools for seen/unseen checks, thread context, preferred sources

### What Thread Does Not Do

- Write directly to Chronicle (proposes to Elephas only)
- Create Taste preferences (proposes for evaluation)
- Browse, replay, or execute actions
- Replace Chronicle, Taste, Corvus, Sift, or Praxis
- Archive raw browser logs (emits only derived signals and summaries)

---

## Quick Start

### Installation

Clone this repository into your skill management system:

```bash
git clone https://github.com/indigokarasu/ocas-thread.git
cd ocas-thread
# Copy to your skills directory
```

### Basic Usage

1. **Snapshot Chrome History** (Mac):
   ```bash
   python3 scripts/snapshot_chrome_history.py
   ```
   Copies locked History database to `/tmp` for safe querying.

2. **Ingest and Normalize**:
   Thread automatically normalizes events to canonical schema (see `references/event_schema.md`).

3. **Reconstruct Sessions and Threads**:
   Events are merged into sessions (time + topical proximity), then sessions are merged into research threads (overlapping entities, domains, query reformulation).

4. **Emit Journals**:
   Three output journals: `thread_activity_journal` (low-level events), `thread_session_journal` (sessions), `thread_research_journal` (primary output with interest candidates and Chronicle proposals).

5. **Propose to Elephas**:
   Research journal includes `chronicle_candidates` field with stable topics and preferred sources.

### Tool Surface

```python
thread.recent_searches(n)        # Last n search queries
thread.recent_visits(n)          # Last n web visits
thread.has_seen(url_or_domain)   # Seen/unseen check
thread.active_threads()          # Currently open research threads
thread.interest_candidates()     # Detected interests with scores
thread.research_summary(topic)   # Summary of research on topic
thread.preferred_sources(topic)  # Highest-affinity domains
```

See SKILL.md for complete tool surface.

---

## Architecture

### Responsibility Boundary

**Thread handles:**
- Chrome History SQLite ingestion via snapshot
- Event normalization to canonical schema
- Dwell-time and engagement classification
- Session reconstruction
- Research thread reconstruction
- Source affinity signaling
- Seen/unseen checks
- Interest candidate detection
- Journal emission (activity, session, research)
- Chronicle candidate proposal

**Thread does NOT handle:**
- Chronicle management (Elephas)
- Preference learning (Taste)
- Pattern discovery (Corvus)
- Search optimization (Sift)
- Task execution (Praxis)

### Ingestion Modes

**Mode 1: Local Near-Realtime (Mac)**
- Snapshot Chrome History every 60–120 seconds using `scripts/snapshot_chrome_history.py`
- Default source: `~/Library/Application Support/Google/Chrome/Default/History`
- Real-time event normalization and journal emission

**Mode 2: Remote Delayed Sync**
- Local exporter on user machine normalizes and pushes events on schedule (suggested: daily)
- Must produce same normalized event schema as local ingestion

**Critical:** Always snapshot before querying. Never read from live locked database.

---

## Package Structure

```
ocas-thread/
├── SKILL.md                          # Operational behavior & tool surface
├── README.md                         # This file
├── LICENSE                           # MIT License
├── references/
│   ├── event_schema.md              # Canonical event schema & field rules
│   ├── session_reconstruction.md    # Session/thread merge heuristics, dwell scoring
│   ├── thread_ontology.md           # Chronicle-aligned node classes & relationships
│   ├── journal_spec.md              # Three journal types with field definitions
│   └── chronicle_boundary.md        # Good/bad candidates, promotion rules
└── scripts/
    ├── snapshot_chrome_history.py   # Safe Chrome History DB snapshot
    └── validate_thread_output.py    # Output journal schema validation
```

### When to Read Each File

| File | When to Read |
|------|-------------|
| `SKILL.md` | First; operational overview and tool surface |
| `references/event_schema.md` | Before normalizing ingested events |
| `references/session_reconstruction.md` | Before computing dwell, sessions, threads, source affinity |
| `references/thread_ontology.md` | When extracting entities or building graph candidates |
| `references/journal_spec.md` | When emitting any journal type |
| `references/chronicle_boundary.md` | When proposing Chronicle candidates |

---

## Journals

Thread emits three structured journal types:

### `thread_activity_journal`
Low-level normalized events with engagement classification. For debugging and dwell-time analysis.

### `thread_session_journal`
Reconstructed sessions with topical focus. For immediate intent grouping and session-level analysis.

### `thread_research_journal` (Primary Output)
Cross-session research threads with stable topics, engagement scores, interest candidates, and Chronicle promotion candidates.

**Full field definitions:** See `references/journal_spec.md`

---

## Interest Detection

Thread flags interests when:
- ≥3 sessions on the topic
- ≥3 long-click events
- High entity/concept overlap
- Behavior spans ≥2 days or session clusters

Flagged interests pass to Corvus and Elephas for evaluation. Thread does not create Taste records directly.

---

## Chronicle Promotion

Thread proposes candidates to Elephas via `research_journal.chronicle_candidates`:

**Good candidates:** Stable topics, durable interests, preferred sources, recurring relationships

**Bad candidates:** Individual URL visits, single-click events, short-duration visits

**Promotion rules and examples:** See `references/chronicle_boundary.md`

---

## Privacy and Data Minimization

- **Ignored:** `chrome://` pages, new-tab pages, extension pages, localhost (unless configured), instant reloads
- **Applied:** Idle dwell noise guard
- **Retention:** Raw events stay local; only derived signals and promotion candidates leave thread

---

## Skill Cooperation

Thread functions independently but cooperates with other skills:

| Skill | Thread Provides |
|-------|-----------------|
| Elephas | Research journals + Chronicle candidates |
| Corvus | Research journals for pattern/novelty detection |
| Sift | Recent thread context for query rewriting |
| Praxis | Seen/unseen status for deduplication |
| Taste | Stable thread signals for preference proposal |
| Mentor | Journals for threshold/heuristic refinement |

---

## Development

### Validate Output

After emitting any journal:
```bash
python3 scripts/validate_thread_output.py <journal_file>
```

Checks schema compliance and field presence.

### Extend Event Types

New event types must:
1. Be added to `references/event_schema.md`
2. Include normalization rules
3. Be added to ingestion logic
4. Pass schema validation

---

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## Contact & Feedback

Thread is maintained as part of the Indigo open-source toolkit. Feedback, bug reports, and pull requests welcome.

---

## See Also

- **Elephas** — Chronicle (memory) management
- **Corvus** — Pattern discovery and novelty detection
- **Taste** — Preference learning and ranking
- **Sift** — Search optimization and query rewriting
- **Praxis** — Task execution and action synthesis
- **Mentor** — Skill refinement and heuristic learning
