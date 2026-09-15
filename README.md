# teach-demo

Interactive, single-page demos of core sequence-analysis algorithms, written for
teaching. Each page takes toy inputs you can edit and lets you step through the
algorithm one action at a time.

| Page | Topic |
|---|---|
| `edit-distance.html` | Edit distance by dynamic programming, with traceback |
| `ond.html` | Ukkonen's O(ND) algorithm in wavefront style |
| `blast.html` | BLAST-like seed-and-extend alignment |
| `bwa-aln.html` | bwa-aln backtracking on a prefix trie, with the D-array bound |
| `ovasm.html` | Genome assembly with an overlap graph: overlap, reduce, compact |

`index.html` only links to the pages above.

## Standalone pages

Every demo page is a **single, self-contained HTML file**: inline CSS, vanilla
JavaScript, no build step, no external libraries, and no dependency on any other
file in this repository. You can copy any one of them elsewhere, open it directly
from disk (`file://`), or embed it in a course site, and it will work unchanged.
Shared helpers are deliberately duplicated in each file rather than factored out,
so that this independence holds.

## Use

Open a page in a browser, or visit the GitHub Pages site built from this
repository. Common keys on every page: `→` step, `←` back, `f` run all, `r` reset;
each page lists its own extras in the hint paragraph under its controls.
