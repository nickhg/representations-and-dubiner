# Filtered merges make deeper representation trees pay: Subset Sum and Meet-LWE

Code and data for the paper by Nick Howgrave-Graham (IACR ePrint, November 2026; link to follow).

Dubiner's bucketing codes, used as *type-class filters* on the representation coordinates, let a merge in a
representation tree avoid generating matched pairs with the wrong split pattern. Their main effect is that
**depth pays**:

| random density-one Subset Sum | plain merges | filtered merges |
|---|---|---|
| HGJ, digits {0,1}           | 0.3371 | 0.3269 |
| BCJ, digits {-1,0,1}        | 0.2891 | 0.2814 (depth 4) |
| BBSS, digits {-1,0,1,2}     | 0.2831 | **0.2690** (depth 5; nothing better found up to depth 9) |

All exponents are log2(time)/n under the usual random-list heuristics. 0.2831 is our recomputation of the
Bonnetain–Bricout–Schrottenloher–Shen exponent. For May's Meet-LWE on ternary keys the filters improve the asymptotic
exponent by 0.013–0.023 for key weights ω ≥ 0.375.

Every number and table in the paper is regenerated from the stored points by two scripts, and each point is
re-costed by an evaluator written separately from the optimiser.

## Requirements

Python 3.9+ with NumPy and SciPy (`pip install -r requirements.txt`). Tested with Python 3.11, NumPy 1.24 and
SciPy 1.10. All scripts can be run from any directory.

## Quick start: reproduce the paper's numbers

```sh
python3 floor/make_numbers.py          # Subset Sum: numbers.tex, depth_table.tex, params_d4..8.tex
python3 ntru_hybrid/make_ml_numbers.py # Meet-LWE: ml_numbers.tex, ml_table.tex
```

Both take a few seconds. Output goes to `paper/`, or to the directory named by the environment variable `OUT`. Each
exponent is re-evaluated from the `.npy` points, never copied from a log. The simulation figures are parsed from the
logs in `floor/runs/` and `ntru_hybrid/runs/`.

Independent checks:

```sh
python3 floor/recost.py floor/runs/polish/gen_m4_5_f_cloud.npy 5     # re-cost the depth-5 optimum (0.2690)
python3 floor/finalize_cloud.py                                    # depths 4-9: model == recost == survival recursion
python3 ntru_hybrid/ml_recost.py ntru_hybrid/points/w0.5_d4_f.npy -1,0,1 4 0.5 f
```

## Layout

### Model and optimisers
| file | what it does |
|---|---|
| `gen_lsf.py` | Representation trees over a general digit set (e.g. {-1,0,1,2}), with BBSS accounting and optional type-class filters at each merge. `Model.lp` gives the time exponent for fixed split and filter laws. |
| `gen_opt.py DIGITS d seed p\|f [init.npy]` | Local optimiser (Powell + Nelder–Mead) over the split and filter laws. |
| `cloud/polish_worker.py RES d id hours [seed]` | Basin-hopping worker used for the large runs: many workers share a results directory. |
| `cloud/seed/` | Seed points for the cloud runs. |

### Subset Sum checks and simulations (`floor/`)
| file | what it does |
|---|---|
| `recost.py point.npy d` | Independent re-cost of a saved {-1,0,1,2} filtered point. Only the parameter layout is shared with `gen_lsf`; the tree, the information terms and the LP are re-derived. |
| `recursive_floor.py DIGITS d VARIANT ...` | Depth-d floors under strict (F0–F2), BBSS (B1, B2) and survival-recursion (S1, S2) accounting. |
| `finalize_cloud.py` | For depths 4–9: checks that the best point has the same exponent under the model (B2), `recost.py` and the survival recursion (S2). |
| `make_numbers.py` | Regenerates every Subset Sum number and table in the paper. |
| `hgj_example.py` | The worked example in the introduction (the top HGJ merge, with and without a filter). |
| `bias_sim.py`, `bias_sim_sparse.py` | Small-n simulations: does a filter bias which parents survive a merge? |
| `survival.py n c [draws] [seed]` | Survival by depth in a real HGJ tree. |
| `sibling_sim.py n k [draws] [seed]` | Joint survival of two planted siblings, each built by its own filtered merge. |
| `runs/` | Best points (`rf_*`, `polish/gen_*`), starting points and logs. Files `rf_{digits}_d{d}_{variant}` hold the result of `recursive_floor.py`. `polish/gen_m4_{d}_f_cloud.npy` are the cloud optima for {-1,0,1,2}. |

### Meet-LWE (`ntru_hybrid/`)
| file | what it does |
|---|---|
| `meetlwe_model.py` | May's Meet-LWE (ePrint 2021/216, §§7–8) as a representation-tree LP, with optional filters and the survival recursion. `meetlwe_model.py check` reproduces May's Tables 5 and 7 at depth 4 (slow: more than 5 minutes). |
| `ml_recost.py point.npy DIGITS d omega [f]` | Independent re-cost of a Meet-LWE point. |
| `may_rep1.py omega` | Re-optimisation of May's Rep-1 with his own formulas. |
| `ml_hop.py`, `ml_worker.py` | Optimisers (single process and shared-directory workers). |
| `ml_fixed_law.py omega d` | Filters added to the best unfiltered point, with the split laws held fixed. |
| `bias_sim_meetlwe2.py` | Filter-bias simulation at the binding filtered merge of the ω = 0.5 optimum. |
| `make_ml_numbers.py` | Regenerates the Meet-LWE numbers and table. |
| `points/w{omega}_d{d}_{p\|f}.npy` | Best plain (`p`) and filtered (`f`) points for each key weight and depth. |
| `runs/` | Logs of the checks and simulations. |

### Point format
A point `z` for depth `d` is a flat array: `d` rows of split logits, then (for filtered points) `d` rows of filter
logits. Within a split row, for each parent digit in ascending order, the logits of its unordered splits after the
first come in sequence; the first split is fixed at 0 and the row goes through a softmax. The full layout is in the
docstring of `floor/recost.py`.

## Not included
The raw outputs of the cloud optimisation runs (`cloud/results/`, about 21 MB of logs) are available on request.
The best points from those runs are included.

## Citation
```bibtex
@misc{HowgraveGraham2026filtered,
  author       = {Nick Howgrave-Graham},
  title        = {Filtered merges make deeper representation trees pay: Subset Sum and Meet-LWE},
  howpublished = {Cryptology ePrint Archive, Paper 2026/XXXX},
  year         = {2026},
}
```

## Acknowledgements
The experiments and much of the code were done with AI assistance (Anthropic's Claude); see the paper.

## License
MIT; see `LICENSE`.
