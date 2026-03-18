# Thread Schemas

## NormalizedEvent
```json
{
  "event_id": "string",
  "event_type": "string — search_query|web_visit|typed_navigation|download|bookmark_add|bookmark_open|form_submission|redirect",
  "timestamp": "string — ISO 8601",
  "url": "string",
  "domain": "string",
  "title": "string|null",
  "search_query": "string|null",
  "referrer_url": "string|null",
  "referrer_domain": "string|null",
  "visit_duration": "number|null — seconds",
  "engagement_signal": "string|null — bounce|short_click|neutral_click|long_click|deep_engagement",
  "device": "string",
  "source": "string — chrome_local|remote_sync",
  "session_hint": "string|null",
  "metadata": "object|null"
}
```

## Session
```json
{
  "session_id": "string",
  "start_time": "string — ISO 8601",
  "end_time": "string — ISO 8601",
  "primary_queries": ["string"],
  "domains_visited": ["string"],
  "long_click_count": "number",
  "downloads": ["string"],
  "candidate_entities": ["string"],
  "candidate_concepts": ["string"],
  "engagement_summary": "object"
}
```

## ResearchThread
```json
{
  "thread_id": "string",
  "topic_label": "string",
  "first_seen": "string — ISO 8601",
  "last_seen": "string — ISO 8601",
  "sessions": ["string — session_ids"],
  "primary_queries": ["string"],
  "entities": ["string"],
  "concepts": ["string"],
  "sources": ["string — domains"],
  "engagement_summary": "object",
  "thread_strength": "number — 0.0 to 1.0",
  "novelty_score": "number — 0.0 to 1.0",
  "interest_candidate": "boolean"
}
```

## SourceAffinity
```json
{
  "domain": "string",
  "topic": "string|null",
  "affinity_score": "number — 0.0 to 1.0",
  "long_click_count": "number",
  "revisit_count": "number",
  "typed_navigation_count": "number",
  "last_seen": "string — ISO 8601"
}
```

## DecisionRecord
Extends shared DecisionRecord from spec-ocas-shared-schemas.md. Thread-specific types: session_reconstructed, thread_merged, interest_detected, chronicle_candidate_emitted, source_affinity_updated.
