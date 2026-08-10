# Futures Hedgecraft

[简体中文](README.md) | **English**

> A futures hedging and risk-sizing skill that converts spot or portfolio exposure into contract count while accounting for multiplier, margin, basis, term structure, roll path, and stress loss.

![type](https://img.shields.io/badge/type-futures--risk-red)
![domain](https://img.shields.io/badge/domain-hedge--roll-maroon)
![license](https://img.shields.io/badge/license-GPLv3-blue)

## What This Is

Futures Hedgecraft is not a direction-calling tool. It translates a market view or hedge objective into contracts, margin usage, roll path, and stress scenarios.

## Project Status and Boundaries

| Item | Statement |
| --- | --- |
| Status | Community Project; not reviewed, certified, or endorsed by QUANTSKILLS |
| Maintainers | Repository maintainers and contributors |
| Data sources | User-provided or user-designated futures prices, settlement data, contract specifications, multipliers, margin, calendars, and delivery rules |
| Assumptions | Contract terms, margin, liquidity, basis, and roll window match the actual trading environment |
| Limitations | Margin changes, limit moves, liquidity decay, basis jumps, and delivery rules can change the risk path |
| Risk boundary | Research and educational example only; it neither fetches data nor executes trades |

## Core Logic

```text
exposure_notional = portfolio_value × hedge_target × beta_adjustment
contract_notional = futures_price × contract_multiplier
contract_count    = round(exposure_notional / contract_notional)
margin_usage      = contract_count × contract_notional × margin_rate
stress_loss       = contract_count × contract_multiplier × adverse_price_move
basis_pnl         = futures_return - spot_return
roll_pnl          = next_contract_price - front_contract_price
trade_ok          = margin_safe + liquidity_ok + roll_plan_defined + stress_loss_acceptable
```

## Quick Start

```bash
python3 scripts/check_test_cases.py
sed -n '1,220p' references/playbook.md
```

## Parameters

| Parameter | Required | Description |
| --- | --- | --- |
| underlying_exposure | yes | Spot or portfolio risk to hedge or express |
| hedge_target | yes | Hedge ratio or target exposure |
| futures_contract | yes | Contract, exchange, expiry |
| contract_multiplier | yes | Contract multiplier |
| futures_price | yes | Tradable futures price |
| margin_rate | yes | Margin requirement |
| roll_window | no | Roll window |
| stress_scenario | no | Gap, limit move, correlation break |

## Validation

Run:

```bash
python3 scripts/check_test_cases.py
```

## Disclaimer

For futures research workflow design. Validate all conclusions against exchange rules, margin notices, and execution constraints.
