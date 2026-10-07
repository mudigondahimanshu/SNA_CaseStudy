# Data Dictionary — v2
**Primary file:** `hn_interactions_v2_CLEAN.csv` — 96,981 rows × 16 columns · **0% missing values**

| # | Column | Meaning | Type | Source | Raw/Derived | Missing |
|---|---|---|---|---|---|---|
| 1 | `record_id` | Sequential ID, ordered chronologically | int | generated | Derived | 0% |
| 2 | `platform` | Constant `hacker_news` | string | generated | Derived | 0% |
| 3 | `story_id` | HN item ID of the submission the exchange occurred under | int | API | Raw | 0% |
| 4 | `story_title` | Submission title (≤200 chars) | string | API | Raw | 0% |
| 5 | `comment_id` | HN item ID of the reply. **Primary key** | int | API | Raw | 0% |
| 6 | `parent_id` | HN item ID of the parent comment | int | API | Raw | 0% |
| 7 | `source_user_hash` | **Influencer** — pseudonymized parent author. **Edge source** | string | derived | Derived | 0% |
| 8 | `target_user_hash` | **Influenced** — pseudonymized replier. **Edge target** | string | derived | Derived | 0% |
| 9 | `parent_username` | Raw username of parent author | string | API | Raw | 0% |
| 10 | `replier_username` | Raw username of replier | string | API | Raw | 0% |
| 11 | `interaction_type` | Constant `reply_to_comment` — every row is a genuine user→user reply | string | derived | Derived | 0% |
| 12 | `thread_depth` | Nesting depth, computed by walking the parent chain (2 → 8+) | int | derived | Derived | 0% |
| 13 | `timestamp_utc` | Reply creation time, ISO-8601 UTC | datetime | API (UNIX) | Raw→normalized | 0% |
| 14 | `comment_excerpt` | Reply text, ≤200 chars, newlines stripped | string | API | Raw | 0% |
| 15 | `source_url` | Permalink to the exact comment — **provenance** | string | derived | Derived | 0% |
| 16 | `retrieval_date` | Collection timestamp, ISO-8601 UTC | datetime | generated | Derived | 0% |

## `hn_edgelist_v2_WEIGHTED.csv`
`source`, `target`, `weight` — 80,759 rows. Aggregated interaction counts (1–23). Direct NetworkX input:
```python
import networkx as nx, csv
G = nx.DiGraph()
for r in csv.DictReader(open("hn_edgelist_v2_WEIGHTED.csv")):
    G.add_edge(r["source"], r["target"], weight=int(r["weight"]))
```

## `hn_edgelist_v2_CORE10.csv`
10-core subgraph — 4,007 nodes / 76,797 edges. Same schema. Use when CELF Monte Carlo runtime matters.

## Fields deliberately NOT included
| Field | Reason |
|---|---|
| `comment_score` | The HN API does not expose comment scores; the Algolia `points` field is `null` for all comments. Verified twice. Omitted rather than shipped 100% empty or imputed. |
| `num_children` | Not reliably available across both collection paths; omitted rather than partially populated. |

## `username_mapping_v2_PRIVATE.json`
Hash → real username. **Not part of the analytical dataset; do not distribute.** Retained solely for verification and reproducibility.
