# 精简任务包

> 用于一个可独立交付的功能或资源批次。Claude 填进原生任务描述，Codex 放在 `tasks/codex/`。不要另存一份相同任务；删掉不适用项。

```yaml
id: T-<nn>
revision: 1
owner: <role>
milestone: M<n>
gdd_refs: [<approved_document_revision>]
depends_on: []
allowed_paths: [<paths>]
base_commit: <actual_sha>
verification_profile: docs # docs | local | integration | milestone
timing_or_performance: false
status: ready
```

## 目标与完成条件

- 目标：〈本次改变什么玩家行为，或交付什么资源〉
- 验收：〈少量可观察结果；已有 GDD 内容直接引用章节〉
- 边界：〈本次不能改变的设计、参数、接口〉

## 本次需要的上下文

| 文件与章节/资源 | 修订或哈希 | 本次为什么需要 |
| --- | --- | --- |
|  |  |  |

已在当前会话读过、修订未变的内容直接沿用。只列相关章节，不列项目的全部文档。遇到矛盾以实际文件和已批准决定为准。

## 执行与验证

- 预期产出：〈文件、资源、演示段落〉
- 本轮检查：〈实际存在的命令或具体观察步骤〉
- 游戏接口（仅运行时任务）：〈连接记录链接、所需状态字段/动作/场景；沿用未变的能力表，重启后确认实例〉
- 升级条件：〈例如改到共享接口则转 integration〉
- 真正阻塞项：〈没有则写无；可自行决定的细节不列为审批项〉

## 完成交接

〈完成时在这里更新一次，或链接一份 handoff；不要复制到两处。〉

- 改动路径与结果：
- 检查结果与证据：
- 未解决问题或下一步：
- commit：〈由 lead 集成时填写，执行者无需回来补录〉
