# btc-cent-forecast-01

Forecast Bitcoin's price 4 hours ahead, 12 times over 48 hours.
Entry for the make.fun bounty `btc-cent-forecast-01` ($250, 100 paid places).

## Model

Damped mean-reversion to two short moving averages, blended:

```
forecast = last + 0.15 * (mean(prices[-120:]) - last)
                 + 0.10 * (mean(prices[-360:]) - last)
```

Four-hour BTC moves are dominated by noise, so the forecast stays close to
persistence (the checkpoint close) and only leans toward the recent mean.
Two anchors keep the pull stable across regimes: the 2h anchor adapts fast
when the market drifts, the 6h anchor holds the slower level.

## Validation (honest, no cherry-picking)

Tested on 388 hourly checkpoints across five disjoint weeks of BTC minute
bars (the kit's practice week, 2026-11-07–11, plus four Coinbase BTC-USD
weeks, 2026-10-05–11-02), and on the 65-case practice window.

| metric | this model | persistence |
|---|---|---|
| pooled MAE, 388 cases (cents) | 36,844 | 37,465 |
| 12-case final-format blocks won | 77.6% (152/196) | — |
| mean block improvement | +2.9% | — |
| practice 65-case window | +3.8% | — |
| official practice p00–p04 | −2.9% (5 cases, noisy) | — |

The edge is thin (about +1.7% pooled) and regime-dependent — in the
slow-drift week of 19–26 Oct the model still won 63% of blocks, in the
bouncy week of 12–19 Oct it won 92%. Variants tested and rejected:
pure momentum (worse everywhere), longer anchors (MA1440/MA4320, far worse),
efficiency-ratio regime gates (worse), volatility scaling (worse), and
AR(1)-on-hourly-returns (no edge). The chosen blend is the average of two
plateau points, not a sharp optimum.

## Run

```bash
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Stdlib only, no network, deterministic (byte-identical output across runs).
One `*.out` per `*.in`, JSON `{"forecasts":[{"case":"<id>","price":<n>}]}`
with the price rounded to 5 decimals.

## License

MIT — see `LICENSE` (SPDX-License-Identifier: MIT).
