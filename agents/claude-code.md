# Claude Code 入口

本项目的核心技能说明是 `../SKILL.md`。当用户请求期货对冲、合约 sizing、移仓、基差/carry、价差或保证金压力分析时，先读取该文件，再按需读取 `../references/playbook.md` 和 `../references/test-cases.md`。

必须同时说明合约乘数、保证金占用、压力损失和移仓/退出计划；可交割合约还要核对通知日和交割路径。
