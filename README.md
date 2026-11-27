# png-squeeze-01 — lossless PNG re-encoder

`main.ts` (Node 22, built-ins only) reads every screenshot under
`/input/<id>/image.png`, decodes it to RGBA, and writes the smallest
exact PNG it can find to `/output/<id>/image.png`.

How it squeezes:

- **Exact forms.** The image is analysed once (opaque? grey? how many
  distinct colours?). It is written as a palette PNG when it holds at
  most 256 colours (palette ordered by count, and again by luminance,
  transparent entries first so `tRNS` stays short), as grey / grey+alpha
  when every pixel is grey, as RGB when every pixel is opaque, and as
  RGBA otherwise. A single fully-transparent value is keyed with `tRNS`
  instead of an alpha channel whenever that is exact.
- **Filters.** Every row's PNG filter is chosen by screening: each fixed
  filter (None, Sub, Up, Average, Paeth), the minimum-signed-sum
  heuristic, the fewest-distinct-byte-pairs heuristic, and a per-row
  brute force that compresses each filter trial with zlib at level 4
  after the 32 KiB already written. Every candidate is weighed by zlib
  at level 6, the smallest wins.
- **Deflate.** The winning filtered stream is written with zlib level 9
  (best of three level-9 settings), inside a PNG holding only the
  chunks the checker reads (IHDR, PLTE, tRNS, IDAT, IEND).

Everything is deterministic: no clock, no randomness, fixed amounts of
work, so two runs write the same bytes. One run over the practice set
takes about 15 seconds on this machine, inside the 60-second wall.

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=practice/input MAKEFUN_OUTPUT_DIR=out node main.ts
```
