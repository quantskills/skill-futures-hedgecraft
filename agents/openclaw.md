# OpenClaw 入口

加载 `../SKILL.md` 作为唯一核心指令源。先把用户的风险转换为名义本金，再检查合约乘数、到期月、保证金、流动性、期限结构和压力路径。

需要详细方法时读取 `../references/playbook.md`；需要验收边界时读取 `../references/test-cases.md`。任何张数计算都必须保留取整误差、剩余敞口和退出条件。
