# Fast makefun-site drop-in

A faster reimplementation of makefun-site 1.0.0, make.fun's own static site
generator, for the make.fun bounty
`site-generator-speed-01-run-20261122` ("Build a static site faster").

`solve.js` exports `run(input)`, which returns exactly the output tree
makefun-site 1.0.0's `build` returns for the same input — every page's
`index.html`, the index and tag pages, `build-report.txt` and each static
file, byte for byte — but faster. `main.ts` is the harness driver: it reads
every case from `/input/cases.json`, answers each with `run`, and writes
`/output/answers.json`.

## How it is faster

The reference is a pipeline of plugins over lexed Markdown tokens and an
interpreted template tree, with a regex and several intermediate objects per
step. This version keeps the exact semantics (front matter, template dialect,
Markdown subset, slug rule, report — the reference's README is the spec and
its code the tie-breaker) but:

- fuses Markdown lexing and rendering into one pass, with no token objects;
- renders inline Markdown by native `indexOf` scans for the next special
  character and bulk-escapes the plain runs between them;
- classifies block lines by their first character before ever running a regex;
- caches `slugify` results per run (each case is answered alone; nothing is
  carried from one case to the next);
- renders templates into one joined string with array accumulation, builds
  each post's card once and reuses it, and counts words without a regex.

No dependencies, no I/O, no clock, no randomness: the output is a function of
the input alone.

## Check

On the kit's practice set (3 cases) and on 18 further cases drawn with the
kit's own generator from six other seeds, the answers are identical to
makefun-site 1.0.0's on every case. On this machine the build runs about
2.2x faster than the house baseline (median ~235 ms vs ~530 ms for the three
practice cases, build only).

## License

MIT. See `LICENSE`.
