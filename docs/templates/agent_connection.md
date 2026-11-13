# Agent 游戏连接记录

模板版本：0.8 | 日期：2026-11-13

> 复制到当前任务或 `handoff/<role>/environment/` 的一个记录中，其他地方只引用。共享接口契约见 [agent_interface](../agent_interface.md)。下面都是待填写规格，不代表已有服务。连接密钥和带密钥的 URL 不写进记录。

## 一、一次填写的接入信息

- 项目 / 当前控制者：〈项目 ID、负责操作该实例的角色〉。
- 启动命令与工作目录：〈真实可执行命令〉。
- 接入方式：〈http / mcp_stdio / mcp_http，先选一种〉。
- HTTP/MCP 地址，或已验证的客户端配置位置：〈不含凭证〉。
- SDK / 协议 / 客户端版本：〈实际值；纯 HTTP 无 MCP 依赖时写 not_applicable〉。
- 凭证获取方式：〈本地凭证文件或环境变量名称，不填值〉。
- 当前运行发现文件：〈.agent/runs/<run_id>/connection.json 的实际路径〉。
- 图像/日志取得方式：〈实际 Agent 能读取的文件、URL 或 MCP 资源/图像返回〉。

## 二、当前实例

| 字段 | 实际值 |
| --- | --- |
| instance_id / 启动时间 | pending |
| source identity / data_revision | pending |
| scene / state_revision / tick | pending |
| capability_revision | pending |
| 实际支持的操作及限制 | pending |
| 是否具备真实渲染能力 | unverified |
| 静音 / 无抢焦点窗口 / 隔离存档与浏览器存储 | unverified |
| 发行构建接口剔除证据 | pending |

启动或重载后重新确认实例；不要仅凭端口、PID 或旧连接可用就沿用。未变的能力表和接入说明不反复读取。并行任务需要独立实例或明确的单一控制者。

## 三、调用示意

下面是项目定义的 `game_act` 参数示意，不是 MCP 协议报文，也不是已执行记录。将 fixture、动作和字段替换为能力表中的实际值。

```json
{
  "instance_id": "<current_instance_id>",
  "command_id": "<unique_command_id>",
  "steps": [
    {
      "route": "ui_pointer",
      "target": "<fixture_button_id>",
      "action": "click"
    }
  ],
  "wait_for": {
    "field": "<observable_state_field>",
    "equals": "<expected_value>"
  },
  "observe_fields": ["<observable_state_field>"],
  "timeout_ms": 5000
}
```

服务需要通过真实输入路径操作按钮，并返回实际结果。命令超时先用 `game_result` 查原 `command_id` 和相关状态；不要换一个 ID 盲目重放。普通操作无需自动截图。

## 四、M0 实际证据

| 步骤 | 实际结果 | 原始输出 / 图像证据 |
| --- | --- | --- |
| 从指定 Agent 启动、连接并确认身份 | unverified | pending |
| 读取指定状态 → 执行动作 → 观察响应 | unverified | pending |
| Agent 取得并查看真实渲染截图 | unverified | pending |
| 关闭该实例并观察退出 | unverified | pending |
| 重启得到新实例，旧实例请求被拒绝 | unverified | pending |
| 隔离默认值与发行构建剔除检查 | unverified | pending |

只在首次接入或相关契约/环境变化时重做相应步骤。普通任务按验证档位检查，不把这张表变成每个任务的重复清单。原始输出保留为文件，交接链接一次即可。
