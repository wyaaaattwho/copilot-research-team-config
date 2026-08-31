# Portable GitHub Copilot Research Team

这是一个纯团队配置仓库，只包含 A0–A5 的角色、模型、权限和协作协议。
它不包含任何项目记忆、实验记录、项目路径、会话或本机状态。

## Agent 与模型

| Agent | 职责 | 模型 | 建议 effort |
|---|---|---|---|
| A0 Lead | 协调、拆任务、决策建议 | Claude Opus 5 | High |
| A1 Theory | 理论、假设、可证伪预测 | GPT-5.6 Sol | XHigh |
| A2 Implementation | 代码实现与测试 | GPT-5.6 Terra | High / XHigh |
| A3 Experiment | 实验执行与证据收集 | GPT-5.6 Terra | Medium / High |
| A4 Reviewer | 独立审查与攻击结论 | Claude Opus 5 | XHigh |
| A5 Knowledge | 按需生成交接和项目文档 | GPT-5.6 Luna | Medium |

## 新服务器安装

```bash
git clone <THIS_PRIVATE_REPOSITORY_URL> copilot-research-team-config
cd copilot-research-team-config
npm install --prefix .tools/copilot-cli @github/copilot
.tools/copilot-cli/node_modules/.bin/copilot login
```

## 在 tmux 中连接 GameAgent 与 Terraria benchmark

```bash
tmux new-session -s terraria-research
cd /path/to/copilot-research-team-config

.tools/copilot-cli/node_modules/.bin/copilot \
  -C "$PWD" \
  --add-dir /path/to/GameAgent \
  --add-dir /path/to/TerrariaBenchmark \
  --agent a0-lead \
  --effort high
```

启动后底栏应显示：

```text
a0-lead · Claude Opus 5 · High
```

tmux 暂时离开：`Ctrl+B`，松开后按 `D`。重新进入：

```bash
tmux attach -t terraria-research
```

## 自动模式

若要自动推进并批准工具调用，但只开放明确加入的项目路径：

```bash
.tools/copilot-cli/node_modules/.bin/copilot \
  -C "$PWD" \
  --add-dir /path/to/GameAgent \
  --add-dir /path/to/TerrariaBenchmark \
  --agent a0-lead \
  --effort high \
  --autopilot \
  --allow-all-tools \
  --allow-all-urls
```

不要把 `~/.copilot`、登录 token 或 session-state 提交到这个仓库。Copilot
会话是每台机器本地的；团队配置通过 GitHub 同步。

