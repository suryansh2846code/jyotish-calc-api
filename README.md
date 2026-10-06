# jyotish-calc-api

A Vedic astrology calculation engine, and a stateless HTTP service that exposes
it as a JSON API.

**AGPL-3.0-or-later.** See [`NOTICE`](NOTICE) for attribution and the list of
changes made here — the engine originates from
[atolat/vedic-calc](https://github.com/atolat/vedic-calc) and copyright in it
remains with its author.

## What's here

| Path | What it is |
|------|-----------|
| `src/vedic_calc/` | The calculation engine — 14 modules, pure functions, Pydantic models |
| `calc-api/` | Stateless HTTP service over the engine. Numbers in, numbers out |
| `tests/` | 481 tests, including cross-validation against raw Swiss Ephemeris |
| `benchmarks/` | Accuracy benchmark against two commercial reference APIs |
| `docs/` | Concepts, architecture, API reference, accuracy methodology |

## Accuracy

The benchmark compares against AstrologyAPI.com (Professional) and Prokerala
across ten charts and five compatibility pairs:

```
1015/1015 checks passing (100%)
```

Reproducible: all current-state assertions are pinned to an explicit
`REFERENCE_DATE`, so two runs on different days give identical results. Run it
with `python benchmarks/accuracy.py` (cached fixtures, no API keys needed).

## Running the service

```bash
cd calc-api
uv sync --extra dev
uv run uvicorn calc_api.main:app --reload --port 8800
```

Then <http://127.0.0.1:8800/docs> for the interactive API explorer — 32
endpoints across natal charts, timing, compatibility and the alternative
traditions (KP, Prashna, Jaimini).

See [`calc-api/README.md`](calc-api/README.md) for the response contract,
deployment, and the two engine quirks callers should know about.

## Running the tests

```bash
uv run pytest tests -q --ignore=tests/comparison   # engine, 481 tests
cd calc-api && uv run pytest -q                    # service, 390 tests
```

## Design notes

**Only `core/ephemeris.py` imports pyswisseph.** Everything else works with
abstract values, which is what makes the astronomical backend swappable.

**The service is deliberately generic.** No auth logic beyond a shared secret, no
business rules, no interpretation — it exposes the engine's public functions and
nothing more. Product logic belongs in whatever consumes it.

**Responses are deterministic.** Same input and same `engine_version` produce
identical output, and no endpoint reads the clock — anything time-dependent takes
its date as an explicit parameter. That is what makes caching safe and tests
reproducible.
