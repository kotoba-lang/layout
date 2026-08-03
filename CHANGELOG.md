# Changelog

## 0.3.0 — 2026-08-03

`achievable-ratio` — the missing step from bytes to time.

`cost` counts bytes and always did so correctly. The 16x-predicted /
2.02x-measured gap recorded in 0.2.0 was never a modelling error; it was a
category error in the reading. A bytes ratio becomes a time ratio only when
BOTH arms are memory-bound, and the moment one arm's per-element loop cost
exceeds its memory cost that arm stops getting faster and the ratio caps.

    per-arm time = max(loop-ns-per-element * n, bytes-fetched / bandwidth)

Fed the two numbers measured on an Apple M1 Max — a 3.03 ns/element loop floor
and 18.5 GB/s observed by one thread — this reproduces the measurement:
predicted 13.8 ms / 6.0 ms against a measured 13.844 ms / 6.066 ms, and an
achievable ratio of 2.30x against a measured 2.28x.

`:both-memory-bound?` is the field that matters: when it is false the bytes
ratio is unreachable no matter how good the layout is, and a plan promising it
is promising something the loop will not allow.

16 tests, 81 assertions.


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
