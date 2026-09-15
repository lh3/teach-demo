# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Interactive teaching demos for a class on DP-based sequence alignment, deployed to GitHub
Pages from the repo root (`index.html` links the demos). Each demo is one self-contained
HTML file with inline CSS and vanilla JS, no build step, no dependencies, and must keep
working when opened as a local `file://` URL.

- `edit-distance.html`: fill the Levenshtein DP matrix by clicking cells/rows; arrows to
  every tied predecessor; optional traceback highlight.
- `ond.html`: Ukkonen's O(ND) in the cell-set "wavefront" form (expand to neighbours, then
  extend along matching diagonals), one step per phase.
- `blast.html`: seed-and-extend over a long reference and a short query (index k-mers,
  look up, gap-free end-to-end extension, best hit).
- `bwa-aln.html`: ungapped bwa-aln (`bwt_match_gap` without indels): prefix trie with SA
  intervals drawn as in the slides, D-array lower bound, priority-stack backtracking, and a
  bowtie-style mode without the bound for comparison.
- `ovasm.html`: toy overlap-graph assembler (exact dove-tail overlaps, Myers transitive
  reduction, optional best-overlap filter, unitig compaction) with a built-in layered graph
  layout (`layoutDAG`, no library) and draggable nodes.

## Developing and checking

There is no test runner or linter. The workflow that has been used:

```sh
open edit-distance.html                      # manual check in a browser

# Pure functions are written DOM-free so they can be pulled out and run in Node:
node -e '
const src=require("fs").readFileSync("ond.html","utf8");
eval(src.match(/function computeDP[\s\S]*?\n  }\n/)[0]);
console.log(computeDP("GATTACA","GCATGCU")[7][7]);'

# Render a given step headlessly (append a setK/setS call before the closing "})();"):
perl -0pe "s/  rebuild\(\);\n\}\)\(\);/  rebuild(); setK(8);\n})();/" ond.html > /tmp/t.html
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --disable-gpu \
  --hide-scrollbars --window-size=1000,820 --screenshot=/tmp/t.png file:///tmp/t.html
```

Keep the model functions (`computeDP`, `computeWaves`, `buildSteps`, `buildIndex`, `calcD`,
`runSearch`, `buildModel` and the ovasm helpers) free of DOM access
so this extraction keeps working.

## Shared page architecture

All demos follow the same pattern; keep new pages consistent with it.

- **Precomputed step list, single source of truth.** On any input change, `rebuild()`
  recomputes everything (full DP, wave snapshots, or BLAST stages) and resets a step index
  (`k` in the DP pages, `s` in `blast.html`). Every visual is derived from the model plus
  that index; nothing is mutated incrementally. Step/Back/Run/Reset just set the index and
  call `render()`.
- **Full re-render.** `render()` rebuilds the SVG/HTML as a string and assigns `innerHTML`.
  Matrices are tiny (inputs capped at 20 chars for DP pages, 80/20 for BLAST), so this is
  fast and keeps the code simple. Hover effects that must not flicker toggle CSS classes on
  cached elements instead of re-rendering (`edit-distance.html` `hoverCell`).
- **Event delegation** on the container: handlers use `e.target.closest('g.cell')` and
  `data-i`/`data-j` (or `data-row`, `data-d`, `data-stage`). Text inside cells has
  `pointer-events: none` so clicks land on the cell group.
- **SVG grid conventions.** Sequence A runs along the columns (index j), sequence B down the
  rows (index i); cell (i, j) sits at `(HDR + j*CELL, HDR + i*CELL)` with `CELL = 44`,
  `HDR = 46`. Arrows are `<line>`s shortened at both ends (`r1`/`r2`) with per-colour
  `<marker>` heads so they never overlap the numbers. `blast.html` uses a 16 px strip with
  the reference as the x-axis header and 1-based reference positions everywhere in the UI.
- **Explanation + status lines** under the matrix describe the current step; hover shows a
  per-cell explanation and mouse-out restores the step's text (`showExplainDefault`).
- **Keyboard shortcuts** are page-global except when focus is in a text input/select:
  → / space step, ← back, `r` reset, `f` run all, plus page-specific letters listed in
  each page's hint paragraph.
- Helpers (`esc`, `clean`, `computeDP`, CSS blocks) are copied between files rather than
  shared, on purpose: each page must stay a standalone file. When changing a shared
  convention, update all pages.

## Deployment notes

GitHub Pages serves the repo root, so new demos go at the root, are linked from
`index.html` with relative paths, and must not rely on a server. Slide PDFs the instructor
drops in (`ond.pdf`, `blast.pdf`) are reference material and are not committed.
