# rk-findings

This repository holds the findings site of the rk run: static pages that the run's container
generates on every cycle and commits here. Nobody edits it by hand, and the next cycle would
overwrite a hand edit anyway.

| Link | What it is |
| --- | --- |
| [Live site](https://jgoetzmann.github.io/rk-findings/) | The pages in `docs/`, served by GitHub Pages. |
| [Overview site](https://jgoetzmann.github.io/rk-overview/) | Start here for what the project is and how it works. |
| [rk-harness](https://github.com/jgoetzmann/rk-harness) | The code that generates these pages. |
| [rk-work](https://github.com/jgoetzmann/rk-work) | The run data the pages are built from. |

## The tabs

| Tab | Page | What it shows |
| --- | --- | --- |
| overview | `index.html` | Epoch status and the three method classes side by side, with a way into each. |
| explicit | `explicit.html` | The scored archive: elites against cycle cost, the elite grids, and the practical problems no search saw measured against the classical anchors. It also carries matched accuracy for three of our own solvers (the champion in Q15, rk4 in Q15 and rk4 in float64); no library solver runs fixed-step Q15, so there is no library row. Each archive cell has its own page (`cell-p*-s*-b*.html`), linked from here. |
| implicit | `implicit.html` | SDIRK lane and side-track measurements, all unscored: the stability scan, the budget ladder, Jacobian cost, lane elites and matched accuracy. Each chart states its own arithmetic: the stability algebra is exact over Fractions, and the lane elites are float64. |
| adaptive | `adaptive.html` | Embedded pairs and step-size controllers: the work-precision sweep, the controller gain map, Q15 cost per attempt, the pair census, lane elites and matched accuracy. |
| validation | `validation.html` | Q15 error on the practical and stiff problems, measured time per step, and where roundoff overtakes truncation. The page exists only when validation, benchmark or falsification data does. |
| research log | `hypotheses.html` | The hypothesis ledger grouped by verdict, with the newest model-written interpretations and literature digests. |
| methodology | `methodology.html` | How the numbers are produced, the cost model, the measurement ledger rules and the glossary. |

## How the pages are made

`rk_harness/sitegen.py` in rk-harness writes every page, and the methodology text comes from
`rk_harness/methodology.py`. The runner calls `sitegen.build` at the end of each cycle, writes
into `docs/` and commits. A watchdog on the host pushes on a timer, and GitHub Pages serves
`docs/` from `main`. The build also deletes pages that an older layout wrote, and cell pages
for cells no longer in the archive. It deletes nothing else.

To change a page, edit `sitegen.py` in rk-harness and rebuild from the `rk-harness` directory
with `RK_WORK_DIR` and `RK_FINDINGS_DIR` set:

```powershell
.venv\Scripts\python.exe -c "from rk_harness import archive, sitegen; from rk_harness.paths import findings_dir; sitegen.build(archive.replay(), findings_dir() / 'docs')"
```

A running container keeps the sitegen it loaded at start and overwrites a host build on its
next cycle, so restart it after the change.

## Rules the pages keep

- The same inputs give byte-identical pages: no wall-clock reads, no randomness, sorted
  iteration.
- No page carries JavaScript. Interactivity is `<details>` and links.
- `sitegen.py` holds a list of overclaiming words. If any page contains one, the whole build
  stops and nothing is written.
- Numbers come from rk-work: the scored archive, `validation/results.json`,
  `benchmark/results.json`, `falsification.json`, the hypothesis ledger, the side-track ledger
  and its artifacts, and the two lane `elites.json` documents. The research log also shows
  literature digests and interpretations as model-written text, and the epoch panel on the index
  reads the run's state files. Lane and side-track numbers carry their arithmetic with them.
- Times are stored in UTC and shown in US Central.

This repository has no CI, on purpose. The container runs the banned-word check over every page
before it writes any of them; the determinism and JavaScript rules live in the harness test
suite, which runs there and not here. A workflow in this repository would fire on every push the
run makes (see `docs/CI.md` in rk-harness).
