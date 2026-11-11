# GDD 示例：《守灯人》

这是一个虚构的小游戏，用来演示 [GDD 写法说明](../../templates/gdd_guide.md) 里的文档结构。目录和真实项目的 `docs/gdd/` 一样：

| 文件 | 内容 |
| --- | --- |
| [one_page.md](one_page.md) | **一页策划：先看这一篇。** 整个游戏在一页里 |
| [systems/relight.md](systems/relight.md) | 续火系统：核心机制的细则，各章节都用到了 |
| [systems/moth.md](systems/moth.md) | 飞蛾与挥灯系统：带状态图和每夜节奏表 |
| [systems/dash.md](systems/dash.md) | 冲刺系统：已废弃的文档长什么样 |
| `images/` | 关键画面和示意图，在 GitHub 上能直接看 |

追踪工具不读取 `docs/examples/`，这里的编号不能写进任何 `gdd_refs`。

## 可以看什么

- **一页策划怎么写**：一句话、关键画面、玩法表、体验、设定、范围、系统与待定，正文约 600 个汉字。
- **一页和细则怎么分工**：一页策划是开工时写的，只写关键数字和规则的大意；容差、特殊情况、完整数值表放进系统策划案。M1 之后每个新系统（比如“界面与流程”）都先写细则，一页策划只更新“系统与待定”。
- **修改记录怎么写**：一页策划 r2 和续火系统 r2，写清了改了哪一块、为什么。
- **图怎么配**：关键画面直接在图上标注；续火和飞蛾用 Mermaid 状态图，续火反馈用带编号的示意图。
- **数值怎么写**：字段名和 `data/` 一致，带单位、范围和生效时机。
- **待定问题怎么写**：写明谁来定、最晚什么时候、阻不阻塞当前里程碑。
- **废弃怎么处理**：冲刺系统保留文件，状态改为 `retired`，并说明原因。

## 附：从策划案拆出任务卡

lead 按文档拆任务，验收条件可以写到章节和条目一级。续火功能的任务卡大致是这样（字段见 `docs/workflow.md` §3）：

```yaml
id: T-07
revision: 1
milestone: M1
owner: gameplay
gdd_refs: [GDD-RELIGHT@r2]
depends_on: [T-02]
allowed_paths: [src/mechanics/, data/tuning.json]
paired_test_task: T-07t
acceptance:
  - GDD-RELIGHT §2.2(3)：按住 interact 满 relight_hold_s，燃料回满。
  - GDD-RELIGHT §2.2(4)(5)：松手后进度按 relight_decay_ratio 回落；挥灯时立即清零。
  - relight_hold_s 和 relight_decay_ratio 能在调参面板里调，生效时机与第四章一致。
out_of_scope: [moth_behavior, vfx]
timing_or_performance: true
verification_profile: integration
open_decisions: []
```

配对的测试卡 T-07t 用同样的 `gdd_refs` 和验收条件。tester 照续火系统“六、验收要点”，在 `tests/regression/mechanics/` 下写测试。
