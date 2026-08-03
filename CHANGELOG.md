# Changelog

## 0.1.0 — 2026-08-03

Initial implementation. AoS / SoA / AoSoA planning against a `machine`
descriptor, with a printed cost model.

- `aos` (declaration order, optional `:reorder?` and `:pad-to-line?`), `soa`,
  `aosoa` with a block width chosen so each sub-array is a whole number of
  lines.
- `cost` / `recommend` — compulsory-line model, returns the losers and the
  model itself. `:cost/streams` breaks ties on the thing a line count cannot
  price.
- `lines-for-strided-run` — the two stride regimes, which is where a layout
  decision actually flips.
- `abi-canonical?` / `abi-guard!` — a reordered or padded layout may not cross
  a Canonical ABI boundary.
- `false-sharing-risk` judged on write stride, so SoA is correctly flagged
  where AoS is not.

14 tests, 72 assertions.
