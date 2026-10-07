# Social Interaction Dataset v3 — Expanded Bounded-Population Design
### Influence Maximization Case Study | Hacker News

**Source:** Hacker News official public API + Algolia HN Search API (no authentication)
**Final size:** 312,913 user→user interactions · 6,434 users · 259,122 weighted edges
**Coverage:** 2025-09 → 2026-09 (13 months) · **Collected:** 2026-09-06

---

## 1. What changed from v2

v3 keeps the **identical methodology and threshold** (K=20) as v2. The only change is that the
collection was expanded: the seed population was snowballed from 290 users to **1,967 users**, and
story coverage from 872 to **1,297 threads**. Nothing about the design, schema, cleaning rules, or
network definition changed — so v2 and v3 are directly comparable.

| Metric | v2 | **v3** | Change |
|---|---|---|---|
| Comments indexed | 318,360 | **920,163** | 2.9× |
| Interactions | 96,981 | **312,913** | **3.2×** |
| Users (nodes) | 4,376 | **6,434** | 1.5× |
| Unique edges | 80,759 | **259,122** | **3.2×** |
| **Edge/node ratio** | 18.45 | **40.27** | **2.2×** |
| **Mean degree** | 36.91 | **80.55** | **2.2×** |
| Median degree | 24 | **30** | — |
| Repeated interactions | 16,222 | **53,791** | 3.3× |
| Weighted edges (w>1) | 10,833 | **36,225** | 3.3× |
| Max edge weight | 23 | **34** | — |
| Avg clustering | 0.0395 | **0.0720** | 1.8× |
| Connected components | 1 (100%) | **1 (100%)** | maintained |
| Degree-1 nodes | 0.0% | **0.0%** | maintained |
| Reciprocity | 0.4256 | 0.4188 | maintained |
| Unique stories | 1,138 | **13,773** | 12× |
| Missing values | 0% | **0%** | maintained |

The important result: **density more than doubled while connectivity held at 100%**. Adding users
did not dilute the network — because new users were selected by how often they already interacted
with the existing population, not at random.

---

## 2. Collection Method (unchanged from v2, expanded in scale)

1. **Seed population** — users identified as active spreaders.
2. **User histories** — up to 400 comments per user from the last 12 months via Algolia
   (`tags=comment,author_<user>`), which returns `parent_id` and `author` directly.
3. **Thread completion** — all comments on the 1,297 stories where the population is most active
   (`tags=comment,story_<id>`).
4. **Snowball expansion (new in v3)** — users appearing most frequently as interaction partners but
   not yet collected were added to the population and their histories fetched. Repeated for
   1,677 additional users.
5. **Edge resolution** — each comment's `parent_id` is looked up in a 920,163-comment index. Both
   endpoints resolve by **dictionary lookup with zero extra API calls**.
6. **Population scoping** — users with ≥20 interactions retained (K=20, same as v2).

---

## 3. Processing Report

| Stage | Count |
|---|---|
| Comments indexed | 920,163 |
| Resolved user→user interactions | 430,560 |
| — removed: self-reply | 2,386 |
| — removed: missing story context | 10 |
| After cleaning | 428,164 |
| — excluded: user activity < K=20 (**design filter, not defect removal**) | 115,251 |
| **Final records** | **312,913** |

The K=20 exclusion is a bounded-population design choice and is reported separately from cleaning
so it is never mistaken for defect removal.

---

## 4. Structural Validation

| Metric | Value |
|---|---|
| Nodes | 6,434 |
| Edges (unique dyads) | 259,122 |
| Edge/node ratio | 40.27 |
| **Mean degree** | **80.55** |
| Median / max degree | 30 / 2,328 |
| Density | 0.006261 |
| Isolated nodes | **0** |
| Connected components | **1 (100%)** |
| Degree-1 nodes | **0.0%** |
| Weighted edges (w>1) | 36,225 |
| Max edge weight | 34 |
| Reciprocity | 0.4188 |
| Avg clustering | 0.0720 |
| Degree assortativity | −0.0092 |
| 10-core subgraph | 6,241 nodes / 257,018 edges |
| Provenance complete | 312,913 / 312,913 (100%) |

### Average degree connectivity (neighbours' mean degree, by node degree)
| Node degree | Neighbours' mean degree |
|---|---|
| k = 3 | 124.8 |
| k = 4 | 149.6 |
| k = 5 | 114.3 |
| k ≥ 50 | **169.4** |

Rising with degree, and high in absolute terms even for low-degree nodes — meaning even peripheral
users are attached to well-connected spreaders. This is the structural property that makes influence
propagate rather than die at the first hop.

### Thread depth
| Depth | Records |
|---|---|
| 2 | 115,580 |
| 3 | 70,920 |
| 4 | 46,597 |
| 5 | 30,204 |
| 6 | 19,120 |
| 7 | 11,943 |
| 8+ | 18,549 |

### Temporal distribution (records per month)
2025-09: 631 · 10: 922 · 11: 1,309 · 12: 1,587 · 2026-01: 4,755 · 02: 7,393 · 03: 13,858 ·
04: 22,856 · 05: 41,984 · 06: 56,835 · 07: 69,898 · 08: 77,695 · 09: 13,190

---

## 5. Network Definition (unchanged)

| Element | Definition |
|---|---|
| **Node** | A Hacker News user (salted-hash pseudonym) |
| **Edge** | Directed `source → target` |
| **Direction** | `source` = parent-comment author (**influencer**); `target` = replier (**influenced**) |
| **Edge weight** | Distinct interactions observed source → target (1–34) |

The *action* is replier → parent, but **influence flows parent → replier**. Stored in influence
direction for direct IC/LT use; raw usernames retained on both sides so direction can be reversed.

---

## 6. Files

| File | Contents |
|---|---|
| `hn_interactions_v3_CLEAN.csv` | **Primary dataset** — 312,913 rows × 16 cols, 0% missing |
| `hn_edgelist_v3_WEIGHTED.csv` | 259,122 weighted edges — direct NetworkX input |
| `hn_edgelist_v3_CORE10.csv` | 10-core subgraph (6,241 nodes) |
| `username_mapping_v3_PRIVATE.json` | Hash → username. **Not for distribution** |
| `validation_report_v3.txt` | Machine-generated validation output |

Schema is identical to v2 — see `DATA_DICTIONARY_v2.md`, which applies unchanged
(`comment_excerpt` truncated to 150 chars in v3 for file size).

---

## 7. Limitations (unchanged from v2 — expansion did not remove them)

1. **A reply is attention, not verified adoption.** IC assumes activation = adoption; on HN a reply
   may signal disagreement. Edges measure "read and responded to." This remains the main construct-
   validity caveat and must be stated in the report.
2. **Temporal skew.** Coverage spans 13 months but is weighted to recent months (Aug 2026: 77,695 vs
   Sep 2025: 631), because user histories are capped at 400 recent comments each.
3. **Selection bias by design.** Population is active users (≥20 interactions) — a bounded-population
   design, not a random HN sample.
4. **No ground-truth influence label.** Propagation probabilities must be modelled (e.g. Weighted
   Cascade p = 1/in-degree), never claimed as observed.
5. **Comment scores unavailable** — HN API does not expose them; Algolia `points` is null for
   comments. Column omitted rather than shipped empty.
6. **One hub with degree 2,328.** Not dominant in share terms, but it will appear in most seed sets
   and should be discussed rather than treated as an artifact.

---

## 8. Research Integrity

Every row derives from an actual API response and carries a permalink. No users, comments,
timestamps or interactions were invented, imputed or duplicated. Unavailable fields were omitted
with reasons documented. The IEEE paper (Li, Fan, Wang & Tan, TKDE 2018) is the theoretical
motivation only — it collected no dataset and did not use Hacker News.
