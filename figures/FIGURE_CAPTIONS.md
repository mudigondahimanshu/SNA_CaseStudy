# Figure captions

**Figure 1** - `fig_A_degree_distribution.png`

Degree distribution of the Hacker News interaction network (n=6,434). Left: histogram showing strong right skew (mean 80.55, median 30). Right: log-log plot with power-law fit.

**Figure 2** - `fig_D_centrality_correlation.png`

Spearman rank correlation between centrality measures. High correlation between degree and k-core indicates they capture similar structural roles.

**Figure 3** - `fig_E_redundancy.png`

Redundancy in the degree heuristic. Left: summed individual spread versus actual joint spread for the top-k highest out-degree users. Right: proportion of influence lost to audience overlap, rising with k - a direct empirical demonstration of submodularity.

**Figure 4** - `fig_E_centrality_vs_spread.png`

Correlation of each centrality measure against simulated influence spread. No measure exceeds rho=0.85, and PageRank performs worst because it tracks in-degree rather than out-degree.

**Figure 5** - `fig1_spread_vs_k.png`

Expected influence spread by seed-selection algorithm under the Independent Cascade model with Weighted Cascade probabilities.

**Figure 6** - `fig2_celf_vs_greedy.png`

Computational cost of CELF versus plain Greedy at equal k. Both return identical seed sets; CELF requires substantially fewer spread evaluations.

**Figure 7** - `fig_all_families.png`

All three algorithm families from the Li et al. taxonomy: simulation-based (CELF), proxy-based (centrality heuristics) and sketch-based (RIS).

**Figure 8** - `fig_detection_vs_k.png`

Early-detection performance versus number of sensors. Left: proportion of significant discussions detected. Centre: mean detection delay. Right: precision.

**Figure 9** - `fig_leadtime_precision.png`

Left: distribution of lead time before a discussion reaches half its activity. Right: coverage versus alert-quality trade-off as sensor count increases.

**Figure 10** - `fig_APP3_validation.png`

Validation against observed cascades. Left: mean arrival position (0 = first to comment). Right: mean number of discussions joined.

**Figure 11** - `fig_APP4_sensor_communities.png`

Distribution of selected sensors across Louvain communities, against proportional expectation.

**Figure 12** - `fig_SUMMARY_panel.png`

Summary of principal results: (a) influence maximization, (b) heuristic redundancy, (c) early detection, (d) cascade validation.

**Figure 13** - `GEPHI_full_network.png`  *(not yet produced)*

Hacker News interaction network (6,434 users, 259,122 weighted edges), ForceAtlas2 layout, nodes coloured by Louvain community and sized by degree.

**Figure 14** - `GEPHI_core_subgraph.png`  *(not yet produced)*

High-degree core of the network, filtered to nodes with degree above threshold, revealing structure obscured in the full view.

**Figure 15** - `GEPHI_sensors_highlighted.png`  *(not yet produced)*

Network with the CELF-selected monitoring sensors highlighted, showing their distribution across the network rather than concentration in a single region.

