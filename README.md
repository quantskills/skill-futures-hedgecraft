# 期货对冲工坊

**简体中文** | [English](README.en.md)

> 期货对冲与风险 sizing 技能：把现货或组合敞口转成期货合约数量，并把合约乘数、保证金、基差、期限结构、移仓和压力损失一起纳入设计。

![type](https://img.shields.io/badge/type-futures--risk-red)
![domain](https://img.shields.io/badge/domain-hedge--roll-maroon)
![license](https://img.shields.io/badge/license-GPLv3-blue)

---

## 这是什么

期货对冲工坊不是方向判断工具，而是一个**把期货交易从观点翻译成合约、保证金和压力路径**的技能。它适合股指期货、商品期货、利率期货或 CTA 风格策略的前置研究，尤其适合处理那些“方向看起来对，但合约机制会先把你打断”的场景。

本技能默认把期货方案拆成四层：

1. **敞口层**：要对冲或表达的风险是什么
2. **合约层**：哪个合约、哪个到期月、乘数和流动性是否匹配
3. **资金层**：保证金、杠杆、压力损失和追加保证金路径
4. **期限层**：基差、contango/backwardation、roll yield 和移仓窗口

## 项目状态与研究边界

| 项目项 | 说明 |
| --- | --- |
| 项目状态 | Community Project；尚未获得 QUANTSKILLS 的审核、认证或背书 |
| 维护者 | 本仓库维护者及贡献者 |
| 数据来源 | 使用者提供或指定的期货价格、结算价、合约规格、乘数、保证金、交易日历和交割规则数据 |
| 核心假设 | 合约规格、保证金、流动性、基差和移仓窗口与实际交易环境一致 |
| 已知限制 | 临时保证金调整、涨跌停、流动性衰减、基差跳变和交割规则会改变风险路径 |
| 风险边界 | 仅用于研究与教育示例；不自动获取数据、不执行交易，结论需结合交易所公告与原始数据独立复核 |

## 核心逻辑

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

## 适用场景

- 股票组合的股指期货对冲
- 商品期货敞口管理
- CTA 风格仓位 sizing
- 日历价差和跨期结构分析
- 基差、carry、contango/backwardation 分析
- 合约移仓、交割和通知日风险管理

## 快速开始

```bash
# 校验测试用例
python3 scripts/check_test_cases.py

# 查看期货研究手册
sed -n '1,220p' references/playbook.md
```

### 推荐调用方式

```text
使用 $futures-hedgecraft 设计一个沪深 300 股指期货对冲方案。权益组合规模 5000 万，目标对冲 70%，需要说明合约张数、保证金占用、基差风险、移仓规则和压力情景。
```

## 参数说明

| 参数 | 必填 | 说明 | 建议 |
| --- | --- | --- | --- |
| underlying_exposure | 是 | 被对冲或表达的现货/组合风险 | 先换成名义本金 |
| hedge_target | 是 | 对冲比例或目标风险暴露 | 不要用主观信心代替 |
| futures_contract | 是 | 品种、交易所、到期月 | 流动性优先 |
| contract_multiplier | 是 | 合约乘数 | 没有乘数不能 sizing |
| futures_price | 是 | 期货价格 | 用可交易价格 |
| margin_rate | 是 | 保证金比例 | 使用保守或交易所上调情景 |
| roll_window | 否 | 移仓窗口 | 在入场前定义 |
| stress_scenario | 否 | 跳空、涨跌停、相关性断裂等 | 至少做一个不利情景 |

## 输出结果

| 输出 | 说明 |
| --- | --- |
| 敞口翻译 | 从组合或现货风险到期货名义本金 |
| 合约选择 | 品种、到期月、乘数、流动性和交易约束 |
| 合约张数 | 对冲比例、取整误差和剩余敞口 |
| 保证金占用 | 初始保证金、压力保证金和资金余量 |
| 基差与期限结构 | 现货、期货、carry 和 roll yield 的影响 |
| 移仓/退出计划 | 什么时候移仓，什么条件下退出 |
| 压力测试 | 最坏情景下亏损、保证金和流动性风险 |

## 目录结构

```text
skill-futures-hedgecraft/
├── SKILL.md
├── README.md
├── README.en.md
├── scripts/
│   └── check_test_cases.py
├── references/
│   ├── playbook.md
│   └── test-cases.md
├── assets/
│   └── futures-hedgecraft.svg
├── agents/
│   ├── openai.yaml
│   ├── claude-code.md
│   ├── cursor-rule.mdc
│   ├── portable-loader.md
│   └── openclaw.md
├── metadata.yaml
└── LICENSE
```

## 运行时入口

| 运行时 | 入口 | 用法 |
| --- | --- | --- |
| Codex | `agents/openai.yaml` + `SKILL.md` | 通过技能名称触发 |
| Claude Code | `agents/claude-code.md` | 读取入口后加载 `SKILL.md` |
| Cursor | `agents/cursor-rule.mdc` | 将规则复制到项目 `.cursor/rules/` |
| Hermes | `agents/portable-loader.md` | 按便携入口加载核心说明 |
| OpenClaw | `agents/openclaw.md` | 按入口加载核心说明 |

许可元数据见 `metadata.yaml`，SPDX 标识为 `GPL-3.0-only`。

## 核心约束

| 约束 | 说明 |
| --- | --- |
| 不能漏合约乘数 | 没有乘数，名义本金和张数都不可信 |
| 不只按预期收益 sizing | 仓位必须受保证金和压力损失约束 |
| 不忽略移仓 | 所有期货头寸都必须说明 roll 或退出 |
| 不混淆对冲和投机 | 对冲看风险覆盖，投机看风险预算 |
| 只述不荐 | 输出研究结构、风险假设与可复核证据 |

## 测试用例

测试用例位于 [references/test-cases.md](references/test-cases.md)，覆盖：

- 权益对冲
- carry 交易
- 移仓窗口
- 保证金冲击
- 价差交易
- 交割风险

运行：

```bash
python3 scripts/check_test_cases.py
```

## 免责声明

本项目用于期货研究流程和风险设计。方案应结合交易所规则、保证金通知和真实成交约束独立复核。
