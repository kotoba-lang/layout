# Changelog

## 0.3.2 — 2026-08-03

`achievable-ratio` now has a real test behind it. Constants measured
independently — loop floor on an L1-resident array, bandwidth on a line-strided
scan of a differently-sized one — predicted a configuration neither touched, to
within 19% (erring pessimistic). Recorded in `:model/calibration` as
`:held-out-test`.

The first attempt missed by 65%, and the fault was the calibration rather than
the model: bandwidth measured with a contiguous f64 sum reports the loop floor
in different units, because 8 bytes per iteration at 0.77 ns caps at 10.4 GB/s.


## 0.4.0 — 2026-08-03 (second entry)

The held-out test `achievable-ratio` was owed.

`loop-floor-ns` measures the loop on a 16 KiB L1-resident array, where memory
is free. `bandwidth-bytes-per-ns` measures a **line-strided** scan of a 45 MiB
array, where the loop is amortised over a whole cache line per touch. Neither
shares a size, stride or element width with the configuration then predicted.

Result: predicted AoS 8.501 ms / SoA 3.084 ms for 4e6 elements of 8 doubles;
measured 6.848 / 2.804. Within 19%, erring toward pessimism.

**It failed first, at 65%, and the fault was the calibration.** Bandwidth had
been measured with a contiguous f64 sum — 8 bytes per iteration at 0.77 ns caps
at 10.4 GB/s, so what came back was the loop floor in different units, and the
model duly over-predicted the AoS arm by 2.9x. Measure bandwidth where the loop
is not the bottleneck.


## 0.3.1 — 2026-08-03

**Correction.** The agreements between `achievable-ratio` and measurement
reported in 0.3.0 are not validations. Both inputs were derived from the two
arms being explained — the loop floor from the candidate, the bandwidth from
the baseline — so the formula returns their ratio by construction wherever
neither term clamps. The model may be right; nothing so far has tested it.
`achievable-ratio`'s docstring now says so, and says what an honest test would
need: an L1-resident loop-floor measurement and a separate streaming-bandwidth
measurement, used to predict a configuration nobody measured.

New calibration point, and this one is a real measurement. The serial
summation loop ran at floating-point add *latency* — 2.87 ns/element for one
load and one add, about ten cycles — because every add waited on the previous
one. Four independent accumulators took it to 0.87 ns and moved the measured
AoS/SoA ratio from 2.12x to 6.63x. **The layout was never what was being
measured; the dependency chain was.** At n=8e6 the same change reached 20.2x,
*above* the byte model's 16x, because a 1 GiB array exhausts TLB reach — the
other direction, and also on the not-modelled list.


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
