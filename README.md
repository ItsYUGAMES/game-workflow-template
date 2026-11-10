# 工作流规范 v0.5

日期：2026-11-10。v0.5 在 v0.4 的基础上去掉了对游戏类型的预设，并加了 GDD 骨架。你确认采用后，它就成为项目规则。

## 相对 v0.4 改了什么（v0.5）

| 改动 | 写进了哪里 |
| --- | --- |
| `combat` 队友改名为 `gameplay`，负责玩家动作和核心机制，不限于战斗；`src/combat/` 改为 `src/mechanics/` | `AGENTS.md`、`CLAUDE.md`、`workflow.md` §1–2、`contracts.md` §1、`assets_and_targets.md` §5 |
| M1 不再写死"移动、闪避、三段攻击、木桩敌人"，改为"核心机制用占位画面可玩" | `workflow.md` §5 |
| M1 前必须批准的 GDD 内容改为通用清单；类型专属的规则由你按需加成新段落 | `workflow.md` §5、`docs/gdd/index.md` |
| GDD 段落新增 `status: draft / approved / retired` 和 `needed_by: <里程碑>`；`gdd:check -- --milestone <Mn>` 列出还没批准的段落 | `traceability.md` §1、§3 |
| 保留测试的 `combat/` 文件夹改为 `mechanics/` | `validation.md` §1 |
| 时钟、数值生效时机、角色动画状态的说明里，攻击相关的内容改成举例，并补上回合制的情况 | `contracts.md` §3–5、`assets_and_targets.md` §2 |
| 新增 GDD 骨架：7 个领域、23 个段落，全部是 `draft` | `docs/gdd/` |

技术栈（Three.js 原型 → Godot 移植）、角色分工、提交与 gate、证据和审查规则都没有变。

## 文件

| 文件 | 内容 |
| --- | --- |
| `AGENTS.md` | 所有 Agent 共用的入口：权限、必读文档、底线规则、完成与里程碑要求 |
| `CLAUDE.md` | Claude Code 专用：角色与模型、启动核对、lead / 队友 / deps 的职责、完成钩子 |
| `docs/workflow.md` | 角色、目录所有权、任务记录、提交与 gate、里程碑 |
| `docs/contracts.md` | 代码分层、core 变更规则、时钟、命名单位、编辑器保存与热更新 |
| `docs/assets_and_targets.md` | 资产规范、Blender、灯光特效、目标包、Godot 交接 |
| `docs/validation.md` | 保留测试、gate 与钩子、完成定义、证据、审查、性能 |
| `docs/traceability.md` | 新增：GDD 编号与 `gdd_refs`、影响分析、提示词归档 |
| `docs/gdd/` | v0.5 新增：通用 GDD 骨架，`index.md` 是入口 |

入口文件和契约文档按你的全局约定用英文写。GDD、任务说明、handoff、审查报告、说明文档可以用中文。

## 相对 v0.3 改了什么

**你的决定**

| 决定 | 写进了哪里 |
| --- | --- |
| 写源码的是三个 Claude 队友 | `AGENTS.md` 权限部分、`workflow.md` §1，并写明它优先于本机的全局默认 |
| 测试由独立的 GPT Agent 负责 | 新增 `tester`（Codex，GPT-6 Astra，high）：写全部保留测试、管 gate 和完成钩子、性能测量工具。实现者不改测试，tester 不改被测源码。需要保留测试的任务，lead 同时开一张配对的测试卡；测试失败时由 lead 转成修复任务交给实现者 |
| Git 提交由 lead 负责 | lead 是唯一的提交者：角色写完 handoff 后告诉 lead，lead 用 `commit:task` 只暂存该任务的路径、核对 diff、提交并带上 `Task: <ID>` 标记，再把 SHA 回报给角色。完成钩子靠这个标记找到提交 |
| 依赖安装交给更低级的模型 | 新增 `deps` 子 Agent（`claude-haiku-5-5`），只改依赖项和锁文件，不提交；由 lead 批准、systems 调用，lead 提交 |
| Claude 以 Agent Teams 任务列表为准，Codex 每张卡一个文件、状态写在卡里 | `workflow.md` §3；去掉了 v0.3 的事件日志和 `codex_queue.md` |
| 完整证据清单只给时序和性能任务 | 任务卡新增 `timing_or_performance` 字段；`validation.md` §4 |
| 只改 `data/` 的提交跑一遍 gate 就够，行为变化才交给 reviewer | `workflow.md` §4、`validation.md` §5 |
| 只保留连招窗口、编辑器冲突这类测试，放在独立文件夹 | 统一放在 `tests/regression/<领域>/`，列明八类可保留的测试；各角色自己的检查放在不入库的 `tmp/probes/`，完成前删除；gate 会检查文件位置 |
| GDD 编号与 `gdd_refs` | 新增 `traceability.md` §1–3：段落编号永不复用、引用带版本号、`gdd:impact` 找出受影响的任务和资产 |
| 提示词归档 | `traceability.md` §4：每次生成（包括被否决的）都记录模型版本、完整提示词、参数、参考图哈希、时间、验收状态 |
| Godot 代码带中文注释，并有图文说明 | `assets_and_targets.md` §7：GDScript 注释用中文；每个系统一份带 Mermaid 图和编辑器截图的中文说明，改代码时同步更新 |

**我额外做的调整**

- **不再暂停写入：** 去掉 v0.3 里"所有人暂停写入"的集成窗口。lead 只提交该任务的路径，gate 在单独的 gate worktree 里对这个提交运行，一次一个排队，所以其他角色可以继续写。
- **systems 减负：** 提交交给 lead，gate 和测试交给 tester，依赖安装交给 deps；systems 保留 core、游戏循环、UI、编辑器框架、组装入口和提交脚本。
- **完成钩子：** 写明只有退出码 2 能拦住任务；超时、崩溃、其他退出码都会放行；退出码 2 但 stderr 是"文件不存在"也会放行。适配器必须自己处理这三种情况。
- **成本相关条款：** 按你"不考虑额度"的要求，删掉了 v0.3 里的成本上限和"费用要问制作人"。

v0.3 中保留不变的部分：代码分层与状态视图、core 的兼容性变更规则、数值生效时机（`next_tick` / `next_action` / `rebuild` / `restart`）、编辑器保存冲突检测、按资产种类区分的规格、目标包的时间字段、性能测量口径、审查报告格式。

## 还没有做的

- 这是规范文档，没有实现提交脚本、gate、钩子适配器、编辑器、`gdd:impact`、deps 定义文件、tester 的角色卡或游戏代码。这些在 M0 由对应角色完成，`validation.md` §7 列出了要演练的步骤。
- 没有在真实环境里验证 Agent Teams 队友能否调用子 Agent。如果不能，按规则由 lead 调用 deps。
- `docs/gdd/` 提供了默认的领域划分，每个项目开工时按需增删。性能测量设备需要你在用到之前定下来。
