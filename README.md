# Agent Skills — 统一技能目录标准

> **一个想法，一份开放标准**：让所有 Agent 共享同一份技能，而不是各自为战。
>
> 版本 0.1.0（草案）· 许可证 [CC0 1.0](LICENSE)

---

## 为什么需要它（The Problem）

现在几乎每个 Agent 都内置了自己的 Skills 平台与**独立的存放路径**：

| Agent | 技能存放路径 |
|-------|-------------|
| WorkBuddy | `~/.workbuddy/skills/` |
| CodeBuddy | `~/.codebuddy/skills/` |
| Claude Code / Codex / Cursor / Trae | 各有各的私有目录 |

这导致三个现实痛点：

1. **重复安装**：同一个技能，你要在 N 个 Agent 里各装一遍。
2. **版本分裂**：同一个技能在不同 Agent 里版本不一致，行为对不上。
3. **碎片化难管理**：技能散落各处，难发现、难去重、难统一升级。

---

## 愿景（The Vision）

系统盘存在一个**统一技能目录**（例如 Windows 上的 `C:\Agent Skills\`）。

一个「**Agent Skills 管理器**」负责从各平台收录技能、支持粘贴链接安装；任何**遵循本标准的 Agent** 直接读取这个目录，无需重复安装。你在管理器里选中某个技能，就能在目标 Agent 中 `@技能名` 直接调用——因为它们读的是同一份文件。

---

## Agent Skills 怎么工作（How It Works）

```
1. 收录   管理器从各平台技能市场聚合技能索引
2. 安装   粘贴技能链接（raw SKILL.md 或 zip），一键装入统一目录
3. 选择   在管理器里浏览 / 搜索，选中你需要的技能
4. 调用   在任意兼容 Agent 里直接 @技能名 使用（Agent 读的是同一个统一目录）
```

> 关于"复制到 Agent"：用户心智里常是"在管理器把技能复制到某个 Agent"。更优雅的标准模型是 **Agent 直接读取统一目录，免去重复拷贝**；管理器"复制"只是其中一种实现。标准以"共享读取"为规范模型。

---

## 给用户的价值

- **一次安装，处处可用**：装一次，所有兼容 Agent 都能 `@` 调用。
- **集中去重、统一升级**：版本唯一，升级一处生效。
- **跨 Agent 便携**：换 Agent 不丢技能。

## 给 Agent 厂商的呼吁

采用本目录标准，让你的用户立刻拥有跨 Agent 的技能库。采纳成本极低——只需**读取统一目录（或尊重 `AGENT_SKILLS_DIR`）**。完整规范见 [`SPEC.md`](SPEC.md)。

---

## 采纳清单（Adoption Checklist）

- [ ] 启动时解析技能目录：`AGENT_SKILLS_DIR` 或规范路径（Windows `C:\Agent Skills\`，macOS `~/Agent Skills/`，Linux `~/.agent-skills/`）。
- [ ] 读取 `skills.json`（或扫描子目录）发现技能，见 [`SPEC.md` §5](SPEC.md)。
- [ ] 将技能的 `name` + `description` 注入触发上下文。
- [ ] 支持用户自定义路径（一致性要求 [`SPEC.md` §8](SPEC.md)）。
- [ ] 在文档 / 市场中标注「兼容 Agent Skills 统一目录标准」并链接本仓库。

---

## 规范与协议为什么分离（重要）

很多人第一反应是：「把统一目录写进开源协议里声明一下」。但这里有个关键的设计取舍：

- **LICENSE 管的是版权授权**。本项目用 **CC0 1.0**（公共领域贡献）：任何人可自由采用、修改、商用，无需署名。
- **技术规范与互操作契约**（"所有 Agent 把技能目录统一，或允许自定义路径"）属于**规范**范畴，写在 [`SPEC.md` §8](SPEC.md) 的「一致性要求」里。

把技术条款塞进 LICENSE 会造成法律效力混淆，且 LICENSE 无法真正约束"Agent 必须读取某目录"这类行为。所以本项目刻意**分离**：版权归 CC0，规范归 SPEC。欢迎就此讨论。

---

## 仓库内容

| 文件 | 说明 |
|------|------|
| [`SPEC.md`](SPEC.md) | 技术标准（**核心**）：目录路径、`AGENT_SKILLS_DIR`、技能包布局、最小字段、注册清单、安装 / 发现协议、一致性要求、安全考量。 |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | 厂商如何采纳、开发者如何提 PR 改进本标准。 |
| [`examples/demo-skill/SKILL.md`](examples/demo-skill/SKILL.md) | 最小合规技能示例，作为厂商落地参照。 |
| [`assets/architecture.svg`](assets/architecture.svg) | 架构示意图。 |
| [`LICENSE`](LICENSE) | CC0 1.0 Universal 全文。 |

---

## 许可证

[CC0 1.0 Universal](LICENSE)。你将本标准为公有无保留地奉献给公共领域，任何人可自由使用，无需署名或授权。

## 状态与参与

草案 v0.1.0。这是一份开放标准，靠社区共建。欢迎开 Issue 讨论、提 PR 完善。让我们一起，把乱成一团的 Skills 收拢到一个目录里。

---

## 致谢与打赏

如果这份「Agent Skills 统一技能目录标准」对你或你的 Agent 产品有帮助，欢迎请我喝杯咖啡 ☕

<p align="center">
  <img src="assets/donate-alipay.jpg" width="280" alt="支付宝收款码" />
  &nbsp;&nbsp;
  <img src="assets/donate-wechat.jpg" width="280" alt="微信收款码" />
</p>

你的支持是我继续维护这份开放标准的动力。
