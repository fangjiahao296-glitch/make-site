# faster static site generator

A faster drop-in for make.fun's makefun-site 1.0.0, written for the
"Build a static site faster, 22 November 2026" bounty
(`site-generator-speed-01-run-20261122`).

`solve.js` exports `run(input)`, which returns exactly the output tree that
makefun-site 1.0.0's `build` returns for one case's input — every page's
`index.html`, the index and tag pages, `build-report.txt` and each static
file, byte for byte. `main.ts` is the harness driver: it reads every case
from `/input/cases.json`, answers each with `run`, and writes
`/output/answers.json`.

## How it is faster

The behaviour is the reference's; the speed comes from doing the same work
with less of it:

- **Markdown inline markup** copies plain runs between marks in bulk and
  escapes each run once, instead of one escaped character at a time.
- **Front matter** finds the closing `---` line by search, so the body is a
  single slice and never split and re-joined.
- **Slugs** are cached (titles, tags and headings repeat across a site), and
  the ASCII fast path skips Unicode normalisation.
- **Templates** are parsed once and rendered into an output array (joined
  once per page); a `for` loop reuses one scope frame and one `loop` object
  for all its items instead of one scope per item.
- **HTML escaping** is a plain scan that returns the original string
  untouched when it holds no special character.
- **Word counting** (for `reading_minutes`) counts whitespace runs without
  building a match array; line trimming is a manual scan, not a regex.

No output is changed in any way. All 3 practice cases are byte-identical to
the reference (verified with the kit scorer), the entry is deterministic
(two runs give byte-identical output), and it runs about twice as fast as the
house baseline on the practice set on this machine.

## Files

- `solve.js` — the generator (dependency-free ES module).
- `main.ts` — the driver the harness runs.
- `LICENSE` — MIT.
