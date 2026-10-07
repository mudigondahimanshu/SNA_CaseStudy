# Early Detection of Emerging Discussions in Online Technical Communities
## CELF-Based Sensor Placement on a Hacker News Interaction Network

---

## 1. Introduction and Context

Information does not spread evenly through a social network. A message seeded with one user may
die immediately while the same message seeded with another reaches thousands. Identifying the
second kind of user, under a fixed budget, is the **influence maximization (IM)** problem, with
applications in viral marketing, public-health messaging and misinformation containment.

Li, Fan, Wang and Tan (IEEE TKDE, 2018) survey the algorithmic landscape of IM, organising methods
into simulation-based, proxy-based and sketch-based families. Their work is a **survey**: it
collected no dataset and evaluated no specific network. This project supplies that missing
empirical component, and then applies the framework to a problem with an operational outcome.

**Research question.** Given a budget of *k* users, which users should an organisation
**monitor** in order to learn about emerging discussions as early as possible?

This inverts the classical IM objective. Rather than seeding a message to maximise spread, we
place *sensors* to minimise detection time - the original application for which the CELF
algorithm was devised (Leskovec et al., 2007).

---

## 2. Network Data Collection and Sources

### 2.1 Source selection

Three candidate platforms were evaluated against structural criteria:

| Source | Outcome |
|---|---|
| **Reddit** | Rejected - unauthenticated API access deprecated (May 2026); no restriction was bypassed |
| **GitHub** (GH Archive) | Rejected - 92% of nodes had degree 1; mean reachable set 0.8 nodes, making IM undefined |
| **Hacker News** | **Selected** - official public API, fully connected graph, timestamped interactions |

Rejecting GitHub on measured structural grounds, rather than convenience, is itself a
methodological result: not every interaction graph is a valid substrate for influence maximization.

### 2.2 Why an original dataset was necessary

The detection objective requires **timestamps on real cascades**. Standard IM benchmarks
(NetHEPT, Epinions, DBLP) are static snapshots without cascade timing. This study is therefore
not reproducible on a benchmark dataset.

### 2.3 Collection method

Bounded-population design using the Hacker News API and the Algolia HN Search API:
a seed population of active users was expanded by snowball sampling to 1,967 users, whose full
12-month comment histories were retrieved, together with complete comment sets for 1,297
discussions. 920,163 comments were indexed, from which user-to-user replies were resolved.

### 2.4 Final dataset

| Property | Value |
|---|---|
| Interactions | 312,913 |
| Users (nodes) | 6,434 |
| Weighted edges | 259,122 |
| Temporal coverage | 2025-09 to 2026-09 (13 months) |
| Missing values | 0% across all 16 columns |
| Provenance | 100% permalink-traceable |

**Node** = a Hacker News user (salted-hash pseudonym).
**Edge** = directed, from parent-comment author (influencer) to replier (influenced).
**Edge weight** = number of distinct interactions observed.

Note on direction: the *action* is replier to parent, but **influence flows parent to replier**,
since the parent's content reached and provoked the reply. Edges are stored in influence direction.

---

## 3. Network Modelling and Methodology

### 3.1 Structural properties

| Metric | Value |
|---|---|
| Nodes / edges | 6,434 / 259,122 |
| Mean degree | 80.55 |
| Median / max degree | 30 / 2,328 |
| Density | 0.006261 |
| Connected components | 1 (100% of nodes) |
| Degree-1 nodes | 0.0% |
| Reciprocity | 0.4188 |
| Average clustering | 0.0720 |

Gephi reports average degree as 40.27 because for directed graphs it reports
in-degree; total degree (in + out) is 80.55. Both conventions are reported here.

### 3.2 Diffusion model

**Independent Cascade** with **Weighted Cascade** probabilities, p(u,v) = 1 / in-degree(v)
(Kempe et al., 2003). The dataset has no ground-truth adoption labels - a reply evidences
attention, not adoption - so propagation probabilities are *modelled*, not measured.

**Possible-worlds formulation.** A naive simulator re-samples edge outcomes on every evaluation,
making sigma(S) noisy: measured error at MC=500 was approximately +/-9 nodes, while the gap between
the 5th and 6th best candidates was about 7 nodes. Under that noise CELF and Greedy select
*different* seeds. We therefore pre-sample possible worlds (live-edge formulation), which makes
sigma deterministic and, as a side effect, approximately 13x faster. This is a correctness
decision, not an optimisation.

### 3.3 Algorithms

All three families from the survey taxonomy:

| Family | Method |
|---|---|
| Simulation-based | Greedy, **CELF** |
| Proxy-based | degree, PageRank, k-core, betweenness, eigenvector, closeness |
| Sketch-based | **RIS** (reverse influence sampling) |

### 3.4 Reproducibility

Random seeds fixed (SEED=42, CRN_SEED=12345). Betweenness and closeness approximated by source
sampling given O(n*m) exact complexity. All metrics computed in Python with NetworkX; network
visualisations produced programmatically with matplotlib using a fixed seed.

---

## 4. Analysis and Interpretation

### 4.1 Degree distribution

Mean degree 80.55 against median 30 indicates strong right
skew. [FILL: power-law alpha from Block A, concentration percentages]

### 4.2 Community structure

8 communities detected by Louvain. Intra-community edges:
31.7%; inter-community: 68.3%.

[FILL: modularity Q value] Modularity below ~0.3 indicates **weak** community structure - the
network behaves as one densely interconnected conversational mesh rather than separable groups.
This is a genuine property, and it constrains the later community-coverage analysis.

### 4.3 Does centrality predict influence?

Correlation of each centrality against simulated influence spread:

- Best predictor: **out_degree** (rho = 0.847)
- Worst predictor: **eigenvector** (rho = 0.655)

PageRank underperforms because it is strongly correlated with **in**-degree (rho = 0.963) but only
weakly with **out**-degree (rho = 0.727). Influence propagates along out-edges; PageRank therefore
ranks by attention received rather than reach, which is the wrong direction on a directed graph.

### 4.4 Redundancy and submodularity

|    |   k |   sum_individual |   joint_spread |   overlap_lost_pct |
|---:|----:|-----------------:|---------------:|-------------------:|
|  0 |   5 |           1381.4 |          808.3 |               41.5 |
|  1 |  10 |           2229.1 |         1027.1 |               53.9 |
|  2 |  20 |           3768.6 |         1323.3 |               64.9 |
|  3 |  30 |           5305.8 |         1515   |               71.4 |

At k=30, **71.4%** of the degree heuristic's
combined reach is redundant - the highest-degree users reach the same audiences. Overlap rises
with k (41.5% at k=5), which is submodularity
measured directly. This explains how a heuristic can correlate well with *individual* influence yet
still select a poor *set*: it has no mechanism to avoid redundant coverage.

### 4.5 Influence maximization results

|   k |   Betweenness |     CELF |   Degree |   Eigenvector |   K-core |   PageRank |   Random |
|----:|--------------:|---------:|---------:|--------------:|---------:|-----------:|---------:|
|   1 |       374.074 |  374.074 |  374.074 |       374.074 |   89.426 |    374.074 |   24.619 |
|   5 |       735.966 |  798.232 |  795.734 |       701.68  |  286.624 |    664.986 |   93.25  |
|  10 |      1012.57  | 1039.07  | 1033.35  |       948.892 |  526.508 |    882.61  |  199.53  |
|  15 |      1169.02  | 1205.8   | 1198.57  |      1128.67  |  773.606 |   1055.45  |  273.34  |
|  20 |      1272.79  | 1329.75  | 1328.2   |      1236.52  |  863.834 |   1175.85  |  353.912 |
|  25 |      1369.08  | 1432.62  | 1430.76  |      1355.21  |  930.594 |   1310.53  |  406.864 |
|  30 |      1435.87  | 1519.93  | 1511.98  |      1441.68  |  999.762 |   1414.41  |  461.453 |

At k=30, CELF achieved spread 1,519.9
(23.6% of the network), versus Degree at
1,512.0.

**CELF vs Greedy.** Identical seed sets: 1. Greedy required
9,990 evaluations against CELF's 2,527 - a
4.0x reduction, confirming that lazy evaluation preserves optimality while
reducing cost, as the survey's theory predicts.

**Sketch-based RIS.** Spread 1,506.5 in 6.7s, reaching
0.991 of CELF's spread - the quality-versus-speed trade-off around which the
survey's taxonomy is organised.

### 4.6 Early detection (the application)

|   k |   CELF-detect |   Random |   Top-reach |
|----:|--------------:|---------:|------------:|
|   1 |         0.089 |    0.009 |       0.089 |
|   5 |         0.27  |    0.042 |       0.268 |
|  10 |         0.386 |    0.083 |       0.362 |
|  15 |         0.461 |    0.111 |       0.422 |
|  20 |         0.521 |    0.141 |       0.464 |
|  25 |         0.571 |    0.163 |       0.503 |
|  30 |         0.612 |    0.185 |       0.541 |

With **20 sensors**:

| Metric | Value |
|---|---|
| Discussions detected | 52.1% of 3,418 |
| Mean detection delay | 5.84 hours |
| Median delay | 2.63 hours |
| Precision | 0.81 |
| False alarms | 431.00 |
| Median lead before half-activity | 2.45 hours |
| Detected before half-activity | 77.6% |

Against a median discussion lifespan of 56.4 hours, and random selection at
14.1% - a 3.7x improvement.

### 4.7 Validation against observed cascades

|    | group                  |   users |   mean_position |   median_position |   pct_in_first_10pct |   mean_discussions_joined |   observations |
|---:|:-----------------------|--------:|----------------:|------------------:|---------------------:|--------------------------:|---------------:|
|  0 | CELF detection sensors |      20 |           0.484 |             0.476 |               10.592 |                    173.25 |           3465 |
|  1 | CELF influence seeds   |      30 |           0.445 |             0.41  |               13.089 |                    142.1  |           4263 |
|  2 | Top-20 out-degree      |      20 |           0.442 |             0.404 |               12.956 |                    156.3  |           3126 |
|  3 | Random users           |      50 |           0.501 |             0.497 |               10.017 |                     23.96 |           1198 |

Influence seeds arrive at mean position 0.445 versus 0.501 for
random users (0.5 = network average), a modest but real earliness effect. Detection sensors arrive
*later* (0.484) but participate in more discussions
(173.2 versus 142.1) - the detection optimiser deliberately
trades earliness for coverage, because breadth dominates the detection objective.

**Interpretation.** High-activity users do not systematically *initiate* discussions; they join at
typical points. The detection advantage derives from breadth of participation rather than
precocity. This refines how "influence" should be read in a reply network and directly addresses
the reply-is-not-adoption limitation.

---

## 5. Visualisation

| Figure | Content |
|---|---|
| `fig_network_full.png` | Full network coloured by Louvain community |
| `fig_network_core.png` | High-degree core, labelled |
| `fig_network_sensors.png` | Selected sensors highlighted |
| `fig_A_degree_distribution.png` | Degree distribution and log-log plot |
| `fig_E_redundancy.png` | Redundancy / submodularity |
| `fig1_spread_vs_k.png` | Influence spread by algorithm |
| `fig_all_families.png` | All three algorithm families |
| `fig_detection_vs_k.png` | Detection rate, delay, precision |
| `fig_leadtime_precision.png` | Lead time and alert-quality trade-off |
| `fig_APP3_validation.png` | Arrival-position validation |

Full captions in `figures/FIGURE_CAPTIONS.md`.

---

## 6. Discussion and Implications

### 6.1 Practical application

An organisation monitoring **20 users** detects 52.1% of
significant discussions with a median delay of 2.63 hours, catching
77.6% before the discussion is half over, at roughly 6 alerts per day of
which 81% are genuine. This is a deployable resource-allocation
decision rather than a ranking.

### 6.2 Methodological implications

1. **Centrality is not influence.** No centrality exceeded rho = 0.85,
   and the most widely used (PageRank) performed worst because of directional mismatch.
2. **Set selection is not ranking.** Up to 71% of a heuristic's
   reach is redundant; only marginal-gain selection avoids this.
3. **Not every graph supports IM.** The rejected GitHub graph had a mean reachable set of 0.8.

### 6.3 Limitations

1. **A reply evidences attention, not adoption.** Section 4.7 partially addresses this but the
   construct-validity gap remains.
2. **52% detection means the remainder is missed.** Presented as a
   cost-benefit curve, not as full coverage.
3. **Temporal skew** - coverage spans 13 months but concentrates in recent months, as user
   histories were capped at 400 recent comments.
4. **Selection bias by design** - a bounded population of active users (>=20 interactions).
5. **Sampling approximations** - betweenness and closeness estimated by source sampling.
6. **Weak community structure** limits the community-coverage analysis.

### 6.4 Future work

Ground-truth validation on a platform with explicit adoption signals; cost-aware sensor placement
where monitoring costs differ; online/streaming detection; virality *prediction* rather than
detection.

---

## 7. Conclusion

We constructed an original 13-month Hacker News interaction network (6,434 users,
259,122 weighted edges, fully connected) and applied the influence-maximization framework
surveyed by Li et al. to an early-detection problem.

CELF and Greedy returned identical seed sets with a 4.0x reduction in
evaluations, confirming the theoretical guarantee. Sketch-based RIS reached
0.99 of CELF's spread far faster. We showed that up to
71% of a degree heuristic's apparent reach is redundant, which
explains why principled selection outperforms ranking by popularity.

Applied to detection, **20 monitored users detect 52.1% of
significant discussions within a median of 2.63 hours** - a
3.7x improvement over random selection - giving a median
2.45 hours of warning before peak activity.

---

## 8. References

[1] Y. Li, J. Fan, Y. Wang and K.-L. Tan, "Influence Maximization on Social Graphs: A Survey,"
*IEEE Transactions on Knowledge and Data Engineering*, vol. 30, no. 10, pp. 1852-1872, 2018.

[2] D. Kempe, J. Kleinberg and E. Tardos, "Maximizing the Spread of Influence through a Social
Network," in *Proc. 9th ACM SIGKDD*, 2003, pp. 137-146.

[3] J. Leskovec, A. Krause, C. Guestrin, C. Faloutsos, J. VanBriesen and N. Glance,
"Cost-effective Outbreak Detection in Networks," in *Proc. 13th ACM SIGKDD*, 2007, pp. 420-429.

[4] C. Borgs, M. Brautbar, J. Chayes and B. Lucier, "Maximizing Social Influence in Nearly Optimal
Time," in *Proc. 25th ACM-SIAM SODA*, 2014, pp. 946-957.

[5] V. D. Blondel, J.-L. Guillaume, R. Lambiotte and E. Lefebvre, "Fast unfolding of communities
in large networks," *J. Statistical Mechanics*, vol. 2008, no. 10, P10008, 2008.

[6] A. A. Hagberg, D. A. Schult and P. J. Swart, "Exploring Network Structure, Dynamics, and
Function using NetworkX," in *Proc. 7th Python in Science Conf.*, 2008, pp. 11-15.

[7] M. Bastian, S. Heymann and M. Jacomy, "Gephi: An Open Source Software for Exploring and
Manipulating Networks," in *Proc. 3rd ICWSM*, 2009.

---

## Appendix

**Code:** `section1_sna_analysis.ipynb`, `section1b_local_metrics.ipynb`, `section1e_block_e.ipynb`,
`influence_maximization_HN.ipynb`, `section2b_ris_sketch.ipynb`, `section3a_detection.ipynb`,
`section3b_validation.ipynb`, `section4b_network_figures.ipynb`

**Data:** `hn_interactions_v3_CLEAN.csv`, `gephi_edges.csv`, documentation in `docs/`

**Integrity:** no data fabricated; unavailable fields omitted with reasons documented; no access
restriction on any platform was bypassed.
