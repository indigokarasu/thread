# 🧵 thread

Reconstructs Chrome browser sessions and research threads. Derives engagement signals and detects emerging interests.

---

## 📖 Overview

Thread ingests Chrome History, normalizes browsing events, reconstructs sessions and research threads, classifies engagement depth, and detects interest candidates. Emits structured activity and research journals for downstream skills.

Thread proposes Chronicle candidates to Elephas and provides context to Corvus, Sift, Praxis, and Taste.

---

## 🚀 Quick Start

### 📦 Installation

Clone into your skill management system:
```bash
git clone https://github.com/indigokarasu/thread.git
```

### 📸 Snapshot Chrome History (Mac)

```bash
python3 scripts/snapshot_chrome_history.py
```

Safely copies locked History database to `/tmp` for ingestion.

### 🛠️ Tool Surface

```
thread.recent_searches(n)         🔍 Last n search queries
thread.recent_visits(n)           🌐 Last n web visits
thread.has_seen(url_or_domain)    ✓ Seen/unseen check
thread.active_threads()           🧵 Open research threads
thread.interest_candidates()      ⭐ Detected interests with scores
thread.research_summary(topic)    📋 Research summary on topic
thread.preferred_sources(topic)   📍 High-affinity domains
```

See SKILL.md for full tool surface.

---

## ⚙️ How It Works

1. **📸 Snapshot** Chrome History via `scripts/snapshot_chrome_history.py`
2. **🔄 Normalize** browser events to canonical schema (searches, visits, navigation, downloads, bookmarks)
3. **🔗 Reconstruct sessions** by merging topically-connected events
4. **🧵 Reconstruct research threads** by merging overlapping sessions
5. **⭐ Detect interests** by scoring stability, recurrence, and engagement
6. **📤 Emit journals** (activity, session, research) with Chronicle candidates

---

## 📊 Output Journals

- **📝 thread_activity_journal** — Low-level normalized events
- **📂 thread_session_journal** — Session bursts with topical focus
- **🎯 thread_research_journal** — Primary output; research threads with interest candidates

---

## ⚙️ Configuration

Read `SKILL.md` for:
- 🔌 Ingestion modes (local near-realtime, remote delayed sync)
- 📋 Normalization rules
- 🔗 Session/thread reconstruction heuristics
- ⭐ Interest detection criteria
- 🔒 Privacy and data minimization

Read `references/` for schemas, ontology, promotion rules, and examples.

---

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.
