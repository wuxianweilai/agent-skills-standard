# Agent Skills 统一技能目录标准（Specification）

> 版本：0.1.0（草案 / Draft）
> 状态：开放讨论中。欢迎通过 Issue / PR 共同完善。
> 许可证：CC0 1.0 Universal（见 `LICENSE`）

---

## §1 范围与目标

本标准定义一个跨 Agent 的**统一技能目录（Unified Skills Directory）**规范，使不同的 Agent 产品可以共享同一份已安装的技能，避免重复安装与版本分裂。

**目标**

1. **互操作（Interoperability）**：任一兼容 Agent 都能读写同一份技能。
2. **可发现（Discoverability）**：技能集中存放，便于索引与搜索。
3. **可移植（Portability）**：用户更换 Agent 不丢失已装技能。
4. **最小采纳成本（Lowest adoption cost）**：复用各 Agent 已有的 `SKILL.md` 事实标准，不另起炉灶。

**非目标（本期）**

- 不规定"管理器"的具体实现、UI 或传输协议细节（仅给出安装协议的最小约定）。
- 不规定技能内部逻辑、模型行为。
- 本期为**提案（草案）**，不含参考实现代码。

---

## §2 规范目录路径

统一技能目录的根路径按操作系统确定：

| 操作系统 | 规范路径 |
|---------|---------|
| Windows | `C:\Agent Skills\` |
| macOS   | `~/Agent Skills/`（`/Users/<user>/Agent Skills/`） |
| Linux   | `~/.agent-skills/`（`/home/<user>/.agent-skills/`） |

> 说明：Linux 下带空格的隐藏目录语义不友好，故采用点目录 `~/.agent-skills/`；macOS 保留与人类可读的 `~/Agent Skills/`。三者逻辑等价，仅路径字符串不同。

### 覆盖规则（关键）

允许通过环境变量自定义路径：

- 变量名：**`AGENT_SKILLS_DIR`**（Windows 下大小写不敏感）。
- 若 `AGENT_SKILLS_DIR` 已设置且非空，**Agent 必须使用它**作为技能根目录。
- 若未设置，则回退到上表对应的规范路径。

---

## §3 技能包布局（Skill Package Layout）

一个技能是一个以 `id` 命名的目录，`id` 采用小写 kebab-case 且全局唯一（如 `pdf-editor`、`agent-reach`）。目录结构沿用现有事实标准：

```
<id>/
├── SKILL.md          # 必填：YAML frontmatter + Markdown 正文
├── scripts/          # 可选：可执行代码（Python / Bash 等）
├── references/       # 可选：按需加载到上下文的文档
└── assets/           # 可选：用于输出的文件（模板、图标等）
```

该布局与 WorkBuddy（`~/.workbuddy/skills/<id>/`）、CodeBuddy（`~/.codebuddy/skills/<id>/`）等现有实现一致，厂商无需改造技能包格式即可接入。

---

## §4 SKILL.md frontmatter 最小字段

`SKILL.md` 的 YAML frontmatter 至少包含：

| 字段 | 必填 | 说明 |
|------|------|------|
| `name` | 是 | 人类可读名称（如 `Agent Reach`）。 |
| `description` | 是 | "何时使用"的描述，建议第三人称（如 "This skill should be used when..."）。 |
| `version` | 推荐 | 语义化版本，如 `1.0.0`。 |
| `author` | 推荐 | 作者 / 维护者。 |
| `homepage` | 推荐 | 来源链接（仓库地址），用于安装溯源与更新检查。 |
| `license` | 推荐 | SPDX 标识符（如 `MIT`、`CC0-1.0`）或 "see LICENSE"。 |
| `tags` | 可选 | 分类标签数组，便于发现。 |

> 实现特定字段（如 `allowed-tools`、`agent_created`、`metadata`）为各 Agent 私有扩展，**可选**；Agent 应**忽略未知字段**，以保障前向兼容。

---

## §5 注册清单 `skills.json`

统一目录根下维护一个注册清单，便于 Agent 免全量扫描即可发现技能：

```json
{
  "schemaVersion": "0.1.0",
  "directory": "C:\\Agent Skills",
  "skills": [
    {
      "id": "demo-skill",
      "name": "Demo Skill",
      "version": "1.0.0",
      "source": "https://github.com/<org>/agent-skills-standard",
      "installedAt": "2026-08-12T19:00:00Z",
      "path": "demo-skill"
    }
  ]
}
```

- Agent **应**优先读取 `skills.json` 做发现（性能更好）。
- 若 `skills.json` 缺失，Agent **可**回退为扫描子目录、逐个解析 `SKILL.md` frontmatter。
- `skills.json` 由安装器 / 管理器维护；多写者场景建议**原子写**（写临时文件后 `rename`）。

---

## §6 安装协议（Install Protocol）

1. **输入**：一个 URL，支持两类：
   - 单个 `SKILL.md` 原始链接 → 作为单文件技能安装（`id` 可由 URL 推导或取自 frontmatter `name` 的 slug）。
   - 归档链接（`.zip` / `.tar.gz`）→ 内含一个技能包目录。
2. **下载并校验**：
   - frontmatter 必须含 `name` 与 `description`，否则拒绝安装。
   - `id` 必须 kebab-case 且唯一；冲突时提示用户或追加后缀。
3. **落盘**：解包到 `<AGENT_SKILLS_DIR>/<id>/`。
4. **注册**：写入 / 更新 `skills.json` 对应条目（含 `installedAt`、`source`）。
5. **幂等**：重复安装相同 `id` + `version` 应跳过；不同版本则覆盖。

---

## §7 发现与加载协议（Discovery & Loading）

- Agent 启动时确定目录：`AGENT_SKILLS_DIR` 或按 §2 的规范路径。
- 读取 `skills.json`，将每个技能的 `name` + `description` 注入可触发上下文（与现有"metadata 常驻上下文"机制一致）。
- 命中触发后，按需加载 `SKILL.md` 正文及 `references/*`（**渐进式披露 / progressive disclosure**，沿用现有三级加载）。
- 本标准**不禁止** Agent 同时加载其自有目录与统一目录；统一目录用于跨 Agent 共享层。

---

## §8 一致性要求（Conformance）—— 关键条款

任何在文档或市场中**标称"兼容 Agent Skills 统一目录标准"**的 Agent，**必须**至少满足以下之一：

- **(a)** 默认从统一技能目录（`AGENT_SKILLS_DIR` 或 §2 规范路径）读取并加载技能；**或**
- **(b)** 尊重用户通过 `AGENT_SKILLS_DIR` 显式指定的自定义技能目录，并将其纳入技能发现路径。

**禁止行为**：以"私有技能目录"为由完全忽略统一目录，且不允许用户配置任何外部技能路径。

**一致性级别（可选）**

- **Level 1（读取）**：能从统一目录发现并加载技能。
- **Level 2（读写）**：在 Level 1 基础上，也能将技能安装 / 写入统一目录。

> ### 为什么本条款写在 SPEC 而非 LICENSE
> 很多人第一反应是"把统一目录写进开源协议里"。但 LICENSE（CC0）只解决**版权授权**问题；技术互操作契约属于**规范**范畴。把技术条款塞进 LICENSE 会造成法律效力混淆，且 LICENSE 无法约束"Agent 必须读取某目录"这类行为。因此：
> - **版权弃权** → `LICENSE`（CC0）。
> - **技术规范与互操作契约** → 本文档 §8。
> 这是本项目的核心设计取舍，详见 `README.md` 的「规范与协议为什么分离」。

---

## §9 版本与更新

- 版本号遵循 [SemVer 2.0.0](https://semver.org/lang/zh-CN/)。
- **去重**：同一 `id` 在统一目录中仅保留一份。
- **升级**：安装器比对 `version`，新版本覆盖旧版本；保留 `homepage` 以便回源检查更新。
- **多 Agent 写入冲突**：建议"最后写入者胜出" + `skills.json` 原子写。

---

## §10 安全考量

- **第三方技能不可信**：`SKILL.md` 可包含指令，`scripts/` 可包含可执行代码。
- 建议在沙箱 / 受限环境中执行技能脚本；首次加载未知技能应**显式征求用户授权**。
- **来源校验**：优先从 `https` + 已知 `homepage` 安装；记录 `source` 供审计。
- 不自动执行网络写操作；遵循最小权限原则。

---

## 附录 A：术语

| 术语 | 含义 |
|------|------|
| Agent | 具备技能系统的 AI 助手产品（WorkBuddy、Claude Code、Codex、Cursor、Trae 等）。 |
| 统一技能目录 | 跨 Agent 共享的技能根目录（§2）。 |
| 技能包（Skill Package） | 以 `id` 命名的目录，含 `SKILL.md`（§3）。 |
| 管理器（Manager） | 负责收录、安装、注册技能到统一目录的程序（本标准不规定其实现）。 |

## 附录 B：与现有实现的兼容

各 Agent 现有的私有技能目录（如 `~/.workbuddy/skills/`、`~/.codebuddy/skills/`）可通过以下方式接入统一目录：

- **软链接**：将私有目录软链到统一目录中的某个 `id` 子目录；或反向，将统一目录软链进私有路径。
- **配置指向**：在 Agent 配置中将技能根路径设为 `AGENT_SKILLS_DIR`。

这确保存量技能无需迁移即可参与共享。
