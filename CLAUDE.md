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
- `rna-seq-em.html`: EM for RNA-seq quantification (Eq. 3 of the course notes, same maths as
  `rna-seq-em.py`): simulate transcripts from shared 300-bp blocks and 1-bp reads (uniform or
  exponential 5'/3' bias, optional within-transcript repeats: one transcript gets a duplicated block that a second
  transcript also carries), collapse reads into hit patterns in an
  editable textarea (lengths are fixed by the simulation), then step through EM iterations with the E-step table, estimates vs truth and
  a convergence chart. EM starts from the count-based estimate (each read split evenly among the
  transcripts it hits, weighted by hit multiplicity), not from a flat 1/n, so unique-only data is
  already at the MLE at iteration 0.
- `hmm.html`: two-state HMM (fair/cheat coin, slide 2 of the HMM lecture; same maths as
  `hmm-coin-em.py`): simulate the hidden coins and throws, or load the slides' 100-throw example.
  The model panel has the state diagram on the left and, on the right, the parameters
  (p(F|F), p(C|C), e(0|C); e(0|F) is fixed at 0.5), the stationary distribution, then the
  simulation and step controls; defaults are written into the inputs at load so browser form
  restoration cannot change the model. The observation is a read-only textarea. With
  "estimate this parameter" unticked, steps walk the scaled forward pass (α̃, scale s_i) left
  to right and the backward pass right to left, drawing α̃(i,C) as a light bar until the
  posterior α̃β̃ replaces it. Ticked, e(0|C) = θ is unknown and each step is one EM iteration
  (posterior sums n₀, n₁ → new θ) with convergence charts. Only the rescaled recurrences are
  implemented (no unscaled α anywhere).
- `msa.html`: progressive MSA with linear gap scores (match/mismatch/gap inputs, default +1/−1/−1).
  Stage 1 aligns every pair (one step each) and sets d = (mismatches + gap columns) / alignment
  length; stage 2 is UPGMA (one step per merge, cluster tables before/after); stage 3a, 3b, … aligns
  the two children of each tree node in merge order. Sequences and profiles go through the same
  `alignProfiles`: σ of two columns is the sum over all symbol pairs (gap/gap = 0), headers stack the
  profile's rows, the child holding the lowest-numbered sequence goes on the columns, and the
  traceback prefers diag, up, left. Examples: the slides' 4 sequences (seq–seq, seq–seq,
  profile–profile) and their first 3 (seq–seq, then seq–profile).
- `li-stephens.html`: diploid Li–Stephens HMM for phasing and imputation (examples from `li-stephens.pdf`, hap3 as
  on slides 2–5). The state is an ordered pair z = a·n + b of panel haplotypes. It uses the standard model, not the
  slides' formulas: each copy keeps its template with probability 1−r or jumps to a uniform panel haplotype, so
  t(a′|a) = (1−r)[a′=a] + r/n, and the pair transition is the product. The emission is a per-allele copying error ε,
  summed over both orderings for an unordered heterozygote; a missing genotype (`?`/N) emits 1. Steps 1..m are the
  forward sweep: scaled α̃ via the O(n²) four-term `mix`, and Viterbi δ̃ rescaled so each column's max is 1. Steps
  m+1..2m are the backward sweep: β̃, the posterior, the Viterbi traceback, and posterior decoding, where ties with
  the mirror state (b,a) go to the larger transition into the next chosen state. Grids per SNP show e,
  α̃→posterior and δ̃. The panel shows the selected path as red (a) and blue (b) lines with the phased output
  (lowercase red = allele differs from its template, bold = imputed).
- `read-phasing.html`: read-based phasing as MEC (§6.2 of `phasing-notes.pdf` with 0/1 errors as in
  `phasing-slides.pdf`). Reads are lines of R/A/- (B[k][i] = 1/−1/0); δ_i is the SNP phase (1 = haplotype 1 carries
  R), σ_k the read phase, q = 1 when a covered base ≠ δ_iσ_k. Each restart: random δ (or the given start haplotype
  for restart 1), then rounds of step 2 (each read takes the phase with fewer errors; ties keep the current phase)
  and step 3 (every SNP whose flip strictly lowers its errors given σ flips at once) until step 3 flips nothing; a
  final step shows the best restart. Cells are tinted by the haplotype that carries the base under δ, so errors are
  cells whose tint differs from the read label colour. `exactMEC` brute-forces 2^(m−1) phases (m ≤ 20) with bit
  masks. Example buttons, left of the read box: Simple (the algorithm slide, started from RRAR), Medium (slide 1,
  the default) and Ladder (error-free reads, MEC 0, where 4 of the 5 seed-1 restarts get stuck). Random
  reset (key `s`, after Reset) draws a new seed and clears the start haplotype, so every restart gets a new initial δ.
- `hic-phasing.html`: Hi-C phasing (§6.4 of `phasing-notes.pdf`, slide `hic.pdf`) with 0/1 errors. Each read is a
  line of R/A/- covering exactly two SNPs; it is consistent when b_ib_jδ_iδ_j = 1. The SNP graph has edge weight
  w_ij = δ_iδ_jW_ij (W_ij = Σ b_ib_j), i.e. consistent minus inconsistent reads under the current phase; flipping a
  set S changes the error count by the weight of the cut (S, rest). Each iteration builds a greedy cut (HapCUT-like)
  from every negative edge as seed: the unplaced SNP with the largest |a − b| joins the side it is more positively
  linked to; the most negative cut is kept, its smaller side flipped; stop when no greedy cut is negative. An exact
  min cut would solve the problem in one flip, hence the greedy cut. `exactHiC` brute-forces 2^(m−1) phases. Up to
  four SNPs use the slide's square layout (1 2 / 3 4), more go round a circle. Examples: Slide (start RRRA,
  reproduces the slide's cut of weight −3), Medium (two rounds to the optimum), Hard (seed 1 stops one error above
  the optimum). Same Random reset button and `s` key as `read-phasing.html`.

## Developing and checking

There is no test runner or linter. The workflow that has been used:

```sh
open edit-distance.html                      # manual check in a browser

# Pure functions are written DOM-free so they can be pulled out and run in Node:
node -e '
const src=require("fs").readFileSync("ond.html","utf8");
eval(src.match(/function computeDP[\s\S]*?\n  }\n/)[0]);
console.log(computeDP("GATTACA","GCATGCU")[7][7]);'

# Render a given step headlessly (append a setK/setS call before the closing "})();"; the
# injected JS must not contain "$(", which Perl interpolates, so use document.getElementById):
perl -0pe "s/  rebuild\(\);\n\}\)\(\);/  rebuild(); setK(8);\n})();/" ond.html > /tmp/t.html
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --disable-gpu \
  --hide-scrollbars --window-size=1000,820 --screenshot=/tmp/t.png file:///tmp/t.html
```

Keep the model functions (`computeDP`, `computeWaves`, `buildSteps`, `buildIndex`, `calcD`,
`runSearch`, `buildModel`, the ovasm helpers, `simulate`, `parseCounts`, `emSteps` in
`rna-seq-em.html`, `stationary`, `hmmGen`, `forwardBackward`, `emSteps` in `hmm.html`, and `parseSeqs`,
`alignProfiles`, `upgma`, `buildModel` in `msa.html`, and `parsePanel`, `parseGeno`, `emissions`, `forwardBackward`,
`viterbi`, `posteriorPath`, `phase`, `buildModel` in `li-stephens.html`, and `parseReads`, `parseHap`,
`bestSigma`, `bestDelta`, `runRestarts`, `exactMEC`, `buildModel` in `read-phasing.html`, and `parseReads`,
`linkMatrix`, `greedyCut`, `bestCut`, `runHiC`, `exactHiC`, `buildModel` in `hic-phasing.html`) free of DOM access
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
drops in (`ond.pdf`, `blast.pdf`, `rna-seq-em.pdf`, `2026_BIG_hmm.pdf`, `li-stephens.pdf`, `phasing-notes.pdf`,
`phasing-slides.pdf`, `hic.pdf`) and the reference scripts
`rna-seq-em.py` and `hmm-coin-em.py` are reference material and are not committed.
