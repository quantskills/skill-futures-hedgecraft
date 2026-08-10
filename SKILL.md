---
name: futures-hedgecraft
description: 当需要设计、审查或排错期货对冲、期货仓位 sizing、合约移仓、基差/carry 分析、日历价差、保证金压力测试或 CTA 风格期货配置时，使用此 skill。适用于股指期货、商品期货、利率期货和跨期价差场景，重点处理合约乘数、名义本金、保证金、期限结构、交割规则和压力损失。
metadata:
  organization: QuantSkills
  organization_url: https://github.com/quantskills
  repository: skill-futures-hedgecraft
  repository_url: https://github.com/quantskills/skill-futures-hedgecraft
  project_type: skill
  collection: quantitative-research
  project_status: community-project
  review_status: unreviewed
  license: GPL-3.0-only
  category: quantitative-finance
---

# 期货对冲工坊

## 项目声明

- 项目类型：Community Project / Skill；当前未声明为官方、认证或生产项目
- 维护者：本仓库维护者及贡献者
- 数据来源：由使用者提供或指定的期货价格、结算价、合约规格、乘数、保证金、交易日历和交割规则数据
- 研究边界：仅用于期货研究与教育示例，不自动获取数据、不执行交易
- 已知限制：结果依赖合约规则、流动性、保证金调整、基差、移仓窗口和压力情景；本技能不能替代交易所公告或经纪商风控规则

## 适用场景

1. 用户需要把股票、商品或组合敞口转成期货合约张数
2. 用户需要判断一个期货对冲方案是否覆盖风险、是否过度对冲
3. 用户需要分析 contango、backwardation、基差和 roll yield 对收益的影响
4. 用户需要设计移仓窗口、退出规则或交割前风险处理
5. 用户提到期货对冲、保证金、合约乘数、移仓、价差交易、CTA、基差或交割风险

## 研究立场

期货不是“带杠杆的现货”。期货头寸有期限、有保证金、有合约乘数、有交割和移仓问题。此 skill 的默认立场是：**方向判断只是输入之一，真正的方案必须同时说明合约、资金、期限和压力路径。**

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

## 输入参数

| 参数 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| underlying_exposure | float/str | 是 | 被对冲或表达的现货/组合敞口 |
| hedge_target | float | 是 | 对冲比例或目标风险暴露，如 0.7 |
| beta_adjustment | float | 否 | beta、相关性、久期或 DV01 调整 |
| futures_contract | str | 是 | 期货品种、交易所和到期月 |
| futures_price | float | 是 | 可交易期货价格 |
| contract_multiplier | float | 是 | 合约乘数 |
| margin_rate | float | 是 | 保证金比例 |
| liquidity_constraint | str/dict | 否 | 成交量、持仓量、买卖价差限制 |
| roll_window | str | 否 | 移仓窗口和触发条件 |
| stress_scenario | str/dict | 否 | 跳空、涨跌停、保证金上调、相关性断裂等 |
| delivery_rule | str | 否 | 是否允许交割、通知日和强制退出规则 |

## 工作流

### 1. 敞口翻译

- 明确要对冲的是金额、beta、久期、DV01、商品数量还是利润敏感度
- 将敞口转成名义本金或风险单位
- 如果是对冲，先说明剩余风险；如果是投机，先说明风险预算

### 2. 合约选择

- 检查合约乘数、tick value、最小变动价位、保证金和交易所规则
- 优先选择流动性最好的可交易合约，而不是理论上最贴近的合约
- 明确到期月、交割规则、通知日和持仓限制

### 3. 仓位 sizing

- 用名义本金计算合约张数
- 说明取整误差和过度/不足对冲
- 计算保证金占用和压力损失
- 如果保证金路径不可承受，降低仓位而不是美化假设

### 4. 期限结构与移仓

- 区分现货收益、期货收益、基差变化和展期收益
- 在 contango 下提醒多头 roll drag；在 backwardation 下提醒多头 roll benefit
- 入场前定义移仓窗口，不等到近月失去流动性

### 5. 压力测试

- 至少测试一个不利价格跳变
- 检查涨跌停导致无法退出的路径
- 检查保证金上调和相关性断裂
- 对价差交易，分别检查两条腿的流动性和滑点

## 输出格式

返回结果必须包含：

1. **敞口目标**：对冲或表达的风险是什么
2. **合约选择**：合约、到期月、乘数和流动性理由
3. **头寸规模**：名义本金、合约张数、取整误差
4. **保证金占用**：初始占用、压力占用和资金余量
5. **基差/期限结构**：carry、roll yield 和 basis 风险
6. **移仓或退出计划**：移仓窗口、触发条件、交割前处理
7. **压力情景**：最坏路径下的损失、保证金和流动性风险

## 验收要求

1. 必须出现合约乘数，否则不能计算张数
2. 必须说明保证金占用和压力损失
3. 必须给出移仓或退出计划
4. 如果讨论 carry，必须区分现货观点和曲线观点
5. 如果讨论价差，必须检查双腿执行
6. 如果合约可交割，必须说明通知日、交割或强制退出风险

## 资源

- 研究手册：[references/playbook.md](references/playbook.md)
- 测试用例：[references/test-cases.md](references/test-cases.md)
- 用例校验：[scripts/check_test_cases.py](scripts/check_test_cases.py)
