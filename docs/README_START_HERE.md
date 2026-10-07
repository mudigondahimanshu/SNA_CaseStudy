# Start Here — Influence Maximization Workflow

Three files, used in this order.

| # | File | Where it goes | Purpose |
|---|---|---|---|
| 1 | `gephi_edges.csv` | Gephi → Import Spreadsheet → **Edges table** | 259,122 weighted directed edges |
| 2 | `gephi_nodes.csv` | Gephi → Import Spreadsheet → **Nodes table** (Append) | 6,434 nodes + readable labels + degree attributes |
| 3 | `influence_maximization_HN.ipynb` | Upload to Google Colab | IC model, CELF, baselines, all figures |
| — | `label_map.csv` | reference | pseudonym → `user_00001` label mapping |

---

## Why the files look like this

**`gephi_edges.csv`** — your source file had lowercase `source,target,weight`. Gephi's importer
looks for **`Source,Target,Weight`** capitalised; with lowercase it treats them as ordinary
attributes and builds **zero edges**. A `Type` column set to `Directed` is also included so the
graph type is unambiguous at import.

**`gephi_nodes.csv`** — node `Id` values match the edge endpoints exactly (otherwise Gephi silently
creates orphans). `Label` is a readable `user_00001` style alias ranked by total degree, because
hash strings like `u_7645c5aa9d5f` are illegible in a rendered figure. Degree columns are included
so you can cross-check Gephi's computed values against the validated dataset.

Verified on generation: **mean degree 80.55**, **max degree 2,328** — both match
`validation_report_v3.txt` exactly.

---

## Gephi quick path

1. `File → Import Spreadsheet…` → `gephi_edges.csv` → **Edges table** → Graph Type **Directed** → New workspace
2. `File → Import Spreadsheet…` → `gephi_nodes.csv` → **Nodes table** → **Append to existing workspace**
3. Statistics panel: Average Degree, Graph Density, Connected Components, **Modularity**
   (use weights ✓, resolution 1.0), **PageRank**, **Network Diameter** (gives Betweenness +
   Closeness — expect 10–30 min, it is not frozen)
4. `Data Laboratory → Export table` → save as **`gephi_node_metrics.csv`** → upload to Colab.
   The notebook picks it up automatically as extra baselines.
5. Layout: **OpenOrd** first, then **ForceAtlas 2** with *Approximate Repulsion* ✓, LinLog ✓,
   Scaling 10, Gravity 1.0. Enable *Prevent Overlap* only in the last 30 seconds.
6. Appearance: colour nodes by **Modularity Class**, size by **PageRank** (4→60, spline).
7. Preview: edge opacity **20–30** (full opacity at 259k edges is an unreadable black blob),
   export **PDF or SVG**, not PNG.

You do **not** need to edit `gephi.conf` — on 24 GB RAM the JVM auto-allocates ~6 GB, which is
far more than this graph needs.

---

## Colab quick path

1. Upload the `.ipynb` at <https://colab.research.google.com>
2. Upload `gephi_edges.csv` (and `gephi_node_metrics.csv` once you have it) via the folder icon
3. Run all cells with `QUICK_TEST = True` — finishes in ~2 minutes and proves the pipeline works
4. Set `QUICK_TEST = False` and re-run for the numbers that go in your report
5. Outputs: `fig1_spread_vs_k.png`, `fig2_celf_vs_greedy.png`, `fig3_seed_overlap.png`,
   `im_results.csv`, `celf_seeds.csv`, `experiment_summary.json`
6. Import `celf_seeds.csv` back into Gephi as a **Nodes table (Append)** → colour by `is_seed`
   → that is your seed-highlight figure

---

## Verified before delivery

This notebook was executed end-to-end against the real graph. Confirmed working:

- Graph loads and matches the validation report on all 7 checks
- Possible-worlds sampler is deterministic
- **CELF and Greedy return identical seed sets** with 2.3× fewer evaluations
- All baselines, all three figures, and all exports produce output

### One real bug found and fixed during testing
The first implementation re-sampled edge coin-flips on every evaluation. Monte Carlo noise
(±9 nodes at MC=500) exceeded the gap between competing candidates (~7 nodes), so CELF and Greedy
selected **different** seeds — which would have made Experiment A irreproducible for a grader.
Replaced with the **live-edge / possible-worlds** formulation (Kempe et al. 2003): sigma became
deterministic *and* ~13× faster. This is documented in the notebook and belongs in your methodology.
