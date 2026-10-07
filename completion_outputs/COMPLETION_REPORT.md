# Completion report: HN influence maximization and early detection

## Research question
Which small set of users should be monitored to detect significant observed discussions early?
The application is sensor allocation; classical influence maximization provides a separate model-based comparison.

## Data and integrity
The supplied dataset contains 312,913 retained reply interactions, 6,434 users,
259,122 weighted directed edges and 13,772 observed discussions.
The observation period is 2025-09-06 14:06:06+00:00 to 2026-09-06 08:05:31+00:00: approximately one year spanning 13 calendar months.
All row-level identity, uniqueness and aggregation checks passed. This proves internal consistency,
not external acquisition provenance. Collection scripts and raw API responses were not supplied.
All new user-level outputs use pseudonyms; the original source files do contain raw usernames.

## Network definitions
Edges point from parent author to replier. Out-degree counts distinct observed repliers to an author's comments;
in-degree counts distinct parent authors the user replied to. Undirected weights explicitly sum reciprocal counts.
Canonical weighted Louvain found 13 communities, with modularity Q=0.1225.
Closeness is exact and direction-labelled; betweenness uses 300 sampled sources. A degree CCDF is reported
without asserting a fitted power law. See network_structure.csv and canonical_node_metrics.csv.

## Historical result reproduced
The old 20-sensor set detects 52.1% of 3,418 outbreaks in the full supplied sample,
with median delay 2.63h and precision 0.805.
These are in-sample results and must not be described as a held-out or prospective evaluation.

## Corrected temporal evaluation
Whole observed discussions are split chronologically at 2026-06-25 05:07:33+00:00 and 2026-08-01 19:29:22.400000095+00:00.
258 boundary-crossing discussions are purged. Candidates, graph baselines and the 36.18h
horizon are derived only from training. The nonnegative early reward is max(0,1-delay/H), averaged across outbreaks.
CELF and greedy select identical first five sensors on the training objective. The primary budget is the
pre-specified k=20; a secondary validation-only rule selects k=30.

| split      |   stories |   outbreaks | first_observed_UTC        | last_observed_UTC         |
|:-----------|----------:|------------:|:--------------------------|:--------------------------|
| train      |      8145 |        1382 | 2025-09-06 14:06:06+00:00 | 2026-06-25 04:50:20+00:00 |
| validation |      2614 |         847 | 2026-06-25 05:15:57+00:00 | 2026-08-01 19:29:00+00:00 |
| test       |      2755 |         988 | 2026-08-01 19:32:11+00:00 | 2026-09-06 08:05:31+00:00 |

At k=20 on the test split, CELF detects 26.8%, compared with Top-reach at
45.6% and a mean of 13.2% across 100 random selections.
Detection within the training horizon is 26.4%; precision is
0.866; median delay among detected outbreaks is 3.16h.
The miss-adjusted mean capped delay is 28.30h.
Rates refer only to retained observed discussion data, not every HN discussion.
The relative results must be reported even when they do not favor CELF: training-objective optimization
does not guarantee superior future detection. This experiment tests that assumption rather than presuming it.

| method                 |   detection_rate |   detection_within_horizon |   precision |   mean_reward |
|:-----------------------|-----------------:|---------------------------:|------------:|--------------:|
| CELF-detect            |           0.2682 |                     0.2642 |      0.866  |        0.2178 |
| Top-reach              |           0.4565 |                     0.4443 |      0.7885 |        0.3688 |
| Out-degree             |           0.3927 |                     0.3877 |      0.8151 |        0.3396 |
| PageRank               |           0.4534 |                     0.4352 |      0.7972 |        0.3627 |
| Betweenness            |           0.4211 |                     0.4018 |      0.7619 |        0.3304 |
| Eigenvector-undirected |           0.3087 |                     0.2996 |      0.8333 |        0.2516 |
| Closeness-out          |           0.3907 |                     0.3856 |      0.8428 |        0.3344 |
| K-core                 |           0.2794 |                     0.2773 |      0.8142 |        0.2308 |

### Generalization across time
Normalized early reward at the same frozen 20-sensor budget:

| method                 |   test |   train |   validation |
|:-----------------------|-------:|--------:|-------------:|
| Betweenness            | 0.3304 |  0.3126 |       0.2778 |
| CELF-detect            | 0.2178 |  0.43   |       0.2191 |
| Closeness-out          | 0.3344 |  0.316  |       0.2877 |
| Eigenvector-undirected | 0.2516 |  0.2988 |       0.2147 |
| K-core                 | 0.2308 |  0.2462 |       0.2312 |
| Out-degree             | 0.3396 |  0.3049 |       0.2383 |
| PageRank               | 0.3627 |  0.2856 |       0.2837 |
| Top-reach              | 0.3688 |  0.3445 |       0.336  |

Differences across these periods may reflect estimation error, changing user participation, discussion
composition and the original sampling design. This descriptive comparison does not identify a causal explanation
and was not used to tune the test model.

### Uncertainty and interpretation
Paired CELF-minus-Top-reach differences and exploratory intervals:

| metric         | resampling                    |   difference |   lower_95 |   upper_95 |   test_week_blocks |
|:---------------|:------------------------------|-------------:|-----------:|-----------:|-------------------:|
| detection_rate | story bootstrap               |      -0.1883 |    -0.2207 |    -0.1579 |                  6 |
| detection_rate | calendar-week block bootstrap |      -0.1883 |    -0.2381 |    -0.1355 |                  6 |
| within_horizon | story bootstrap               |      -0.1802 |    -0.2115 |    -0.1488 |                  6 |
| within_horizon | calendar-week block bootstrap |      -0.1802 |    -0.2346 |    -0.1272 |                  6 |
| early_reward   | story bootstrap               |      -0.1511 |    -0.1783 |    -0.1235 |                  6 |
| early_reward   | calendar-week block bootstrap |      -0.1511 |    -0.1901 |    -0.1123 |                  6 |

Story resampling may understate dependence; calendar-week resampling has only 6 blocks.
Delay comparisons on commonly detected outbreaks are in common_detected_delay_comparison.csv.
Sensitivity checks use validation only. No test result is used to retune the frozen sensor selection.
The alert ledger is an offline replay with eventual size labels, not a live prediction system.

## Influence maximization
Primary propagation is standard, uncapped Weighted Cascade: p(u,v)=1/in-degree(v).
CELF uses all 6,434 candidates and 200 fixed selection worlds. Greedy and CELF select
identical first five seeds. RIS uses 20,000 RR sets. All seed sets are evaluated on the same
500 independent live-edge worlds. At k=30, mean spread is CELF=1511.43, RIS=1495.87,
and out-degree=1515.33. The paired Monte Carlo interval must accompany small differences.
CELF is not guaranteed to beat a structural baseline on independent worlds after optimizing a finite sample.
Computational timing separates reusable preprocessing from selection; evaluation is separate.

| method                 |   spread |   mc_lower95 |   mc_upper95 |
|:-----------------------|---------:|-------------:|-------------:|
| CELF                   | 1511.43  |     1497.24  |     1525.63  |
| RIS                    | 1495.87  |     1481.45  |     1510.29  |
| Out-degree             | 1515.33  |     1500.96  |     1529.69  |
| PageRank               | 1414.85  |     1398.85  |     1430.85  |
| Betweenness            | 1431.97  |     1416.81  |     1447.12  |
| Eigenvector-undirected | 1435.88  |     1420.16  |     1451.59  |
| Closeness-out          | 1512.73  |     1498.24  |     1527.22  |
| K-core                 |  902.558 |      880.331 |      924.785 |
| Saved-CELF-capped-WC   | 1510.27  |     1495.93  |     1524.61  |

The classical greedy guarantee concerns the stated monotone submodular objective; here the directly optimized
objective is a finite-world estimate. A fixed 20,000-RR-set implementation is not claimed to have a calibrated
epsilon/delta guarantee. Probability-model sensitivity is reported separately in probability_sensitivity.csv.
The earlier alternate CELF run recorded 0/10 seed overlap; probability robustness must not be claimed.
Set redundancy is measured directly for CELF and degree; redundancy alone does not establish a large
advantage over a competing set. Centrality correlations are against simulated reach, not observed adoption.

## Completed artifacts
Seven PNG figure groups, two self-contained interactive HTML files, canonical graph metrics,
training sensor rankings, a held-out alert ledger, comparison tables, uncertainty tables, sensitivity tables,
claims reconciliation and a SHA-256 provenance manifest are provided in this folder.

## Remaining limitations
| item                                                                       | status                             | needed_for                                                                      |
|:---------------------------------------------------------------------------|:-----------------------------------|:--------------------------------------------------------------------------------|
| Original collection and cleaning scripts, raw API responses and audit logs | Not present in supplied project    | Recollection and independent verification of acquisition/cleaning counts        |
| Earlier Reddit/GitHub attempts and HN v1/v2 datasets                       | Narrative only in supplied project | Verification of platform rejection and earlier-version statistics               |
| Prospective monitoring data / unbiased sampling frame                      | New data required                  | Future operational validation and removal of original population-selection bias |
| Ground-truth adoption or causal influence labels                           | Not observed                       | Claims about adoption/causality rather than reply attention                     |
| Faculty rubric, final formatting requirements and author details           | Not supplied                       | Final submission formatting and rubric compliance                               |
| Browser interaction testing of generated HTML                              | Not claimed by notebook            | Visual/interaction QA in the presentation browser                               |

The full-period activity-filtered sampling frame is already baked into the supplied data. A chronological split
cannot remove that selection bias. Story starts are first retained replies; true starts and complete cascades
are unavailable. Purging can bias duration composition, and test-end censoring can affect eventual sizes.
Replies do not establish adoption, agreement or causality. Arrival-position differences are descriptive,
not statistical proof that an algorithm identified causal influencers. No live deployment or perfect future
accuracy is claimed.

## Primary methodological references
- Kempe, Kleinberg and Tardos (2003), [Maximizing the Spread of Influence through a Social Network](https://www.cs.cornell.edu/home/kleinber/kdd03-inf.pdf).
- Leskovec et al. (2007), [Cost-effective Outbreak Detection in Networks](https://www.cs.cmu.edu/~guestrin/Publications/KDD2007/kdd2007.pdf).
- Borgs et al. (2014), [Maximizing Social Influence in Nearly Optimal Time](https://www.microsoft.com/en-us/research/publication/maximizing-social-influence-nearly-optimal-time/).
- Li, Fan, Wang and Tan (2018), Influence Maximization on Social Graphs: A Survey, IEEE TKDE 30(10), 1852–1872: the original project's theoretical framing, not the source of this dataset.
