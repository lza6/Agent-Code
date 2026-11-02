# Agent-Code

一个专门给 Claude Code 赋能的工具清单目录。以下工具均在本机（Windows）**实际安装并验证可用**，按能力分类整理，含安装/更新命令、使用方法、优缺点与 Agent 自动化建议。

---

## 目录

- [一、代码图谱类（让 Agent 看懂整个仓库）](#一代码图谱类)
- [二、持久记忆类（跨会话记住一切）](#二持久记忆类)
- [三、上下文省 Token 类](#三上下文省-token-类)
- [四、MCP 服务器](#四mcp-服务器)
- [五、Agent 自动接入方式](#五agent-自动接入方式)
- [六、推荐组合与落地建议](#六推荐组合)

---

## 一、代码图谱类

### 1. graft ⭐ 强烈推荐（最好用）

**仓库**：https://github.com/nanonets/graft ｜ **安装/更新**：`npm install -g @nanonets/graft`

把仓库构建成「链接式 Markdown 上下文图谱」+ 每文件卡片，专为 AI 编码 Agent 设计。

```powershell
graft init                 # 初始化（生成 graft/ 目录）
graft build                # 基础构建（$0 免费，无需 API key）：接线图 + 文件卡片
graft build --deep         # LLM 深度模式：概念图谱 + 逐符号摘要（需 key）
graft ask "登录逻辑在哪"    # 查询图谱，返回排序节点 + 精确 file:line
graft map                  # Token 预算内的仓库定向（目录聚类 + 热点）
graft skeleton <file>      # 只看文件签名，了解 API 面最便宜的方式
graft callers <symbol>     # 谁调用了这个符号（--depth all = 完整影响半径）
graft blast                # 一份 diff 的爆炸半径分析，可直接出 PR 评论（Mermaid 图）
graft grep <pattern>       # 按耦合度排序的正则搜索
graft mcp                  # 以 MCP 方式暴露给 Claude Code（已接入）
graft check                # CI 用：图谱过期即失败
```

- **优点**：基础模式完全免费；输出精确到 file:line，Agent 不用盲翻文件；`blast`/`callers` 改代码前就知道影响范围；自带 `init` 可一键接入 Claude Code / Cursor / Codex CLI 等多种 Agent。
- **缺点**：`--deep` 模式需 LLM key；图谱要随代码更新（可 CI 里用 `graft check` 兜底）。
- **Agent 自动化**：✅ 已作为 MCP 接入（`graft_find_code` / `graft_trace_calls` / `graft_repo_map` 等工具，Agent 自动调用）。

### 2. codegraph

**安装**：`npm install -g @nanonets/codegraph`（示例命令）｜ **项目内使用**：

```powershell
codegraph init -i          # 初始化并建索引（生成 .codegraph/）
codegraph index            # 重建索引
codegraph sync             # 增量同步变更
codegraph query "符号名"    # 搜符号
codegraph explore "模块"    # 一次拿到相关符号源码 + 调用路径
codegraph context "任务描述" # 为一个任务构建精准上下文
codegraph impact <symbol>   # 改这个符号会影响什么
codegraph affected <files>  # 改动后哪些测试受影响
codegraph install           # 安装 MCP 到 Claude Code 等 Agent
```

- **优点**：AST 级符号索引，`explore`/`context` 一次调用顶十几次 grep+read；`affected` 精准圈定回归测试范围。
- **缺点**：写入后索引有 ~500ms 延迟；大仓库首次索引稍慢；需在项目里 `init`。
- **Agent 自动化**：✅ 已接入 MCP，CLAUDE.md 中写了优先级规则（结构性问题走图谱，字面问题走 grep）。

### 3. code-review-graph ⚠️（当前故障）

审查知识图谱，`build` 后可查变更影响半径。**本机状态**：`serve` 连接失败（pyvenv.cfg 问题，Python venv 损坏）。修复方向：重装其 Python 运行环境。修复前不推荐作为依赖。

---

## 二、持久记忆类

### 4. icm (Infinite Context Memory) ⭐ 强烈推荐

**仓库**：https://github.com/rtk-ai/icm ｜ **安装/更新**：

```powershell
irm https://raw.githubusercontent.com/rtk-ai/icm/main/install.ps1 | iex   # 安装
icm init --mode all        # 接入 Claude Code（hooks 自动记忆）
icm upgrade                # 升级（带 SHA256 校验）
```

**核心用法**：

```powershell
icm recall "查询词"                    # 语义搜索记忆
icm recall-context "查询词" --limit 5  # 输出格式化好、可直接注入 prompt
icm store -t errors-resolved -c "描述" -i high -k "关键词"   # 存记忆
icm memoir create -n "知识库名" -d "描述"                    # 永久知识图谱
icm memoir add-concept -m "知识库名" -n "概念" -d "定义" -l "type:decision"
icm memoir link -m "知识库名" --from "A" --to "B" -r depends-on
icm learn                              # 扫描项目，把结构存成 Memoir
icm wake-up                            # 输出关键事实包，注入系统提示
icm briefing                           # 编译项目记忆为唤醒简报（缓存零延迟）
icm doctor / backup / consolidate-all  # 体检 / 备份 / 合并压缩
```

- **优点**：hooks 自动存取（会话开始自动 recall、过程自动 store），Agent 无需手动触发也能积累记忆；Memoir 是结构化知识图谱，比平铺记忆强得多；支持时间衰减、压缩合并、导入导出；本地 SQLite，数据不出本机。
- **缺点**：需要配好触发纪律（什么该存什么不该存），否则记忆库会噪音化；LLM 后台提取是可选配置。
- **Agent 自动化**：✅ 全自动（SessionStart/SessionEnd hooks）+ MCP 已接入。

### 5. claude-mem（可选补充）

**仓库**：https://github.com/thedotmack/claude-mem ｜ **安装**：`npx claude-mem install`

跨会话记忆，偏"会话摘要"路线。与 icm 功能重叠，**二选一即可**（本机主力用 icm）。优点是开箱即用；缺点是结构化能力不如 icm 的 Memoir。

---

## 三、上下文省 Token 类

### 6. rtk（RTK — 输出压缩代理）

**命令输出先压缩再进上下文**，保留全部信号、去掉噪音。

```powershell
rtk ls / rtk tree          # 目录列出（token 优化版）
rtk read <file>            # 智能过滤读文件
rtk git log / rtk git diff # 压缩版 git 输出（diff 只留变更行）
rtk test                   # 只显示失败
rtk err <cmd>              # 只显示错误/警告
rtk json <file>            # 只看 JSON 结构不看值
rtk log <file>             # 去重过滤日志
```

- **优点**：长命令输出（构建、测试、日志）直接砍掉 70-90% token，长会话不容易爆上下文；`rtk bypass <cmd>` 可随时拿原始输出兜底。
- **缺点**：压缩有概率丢失细节，排查诡异问题时记得 bypass。
- **Agent 自动化**：✅ 通过全局 `RTK.md` 指令让 Agent 默认走 rtk 输出。

---

## 四、MCP 服务器

**统一管理**：`claude mcp list`（健康检查）｜ **新增**：`claude mcp add <name> --scope user -- cmd /c npx -y <package>`

| MCP | 状态 | 用途 | 接入命令 |
|-----|------|------|---------|
| `graft` | ✅ | 代码图谱查询（find_code / trace_calls / repo_map） | `claude mcp add graft --scope user -- cmd /c npx -y @nanonets/graft mcp` |
| `codegraph` | ✅ | 符号索引 + 影响分析 | `codegraph install`（交互选择 Agent） |
| `icm` | ✅ | 持久记忆 | `icm init --mode all` 自动配好 |
| `sequential-thinking` | ✅ | 复杂问题分步推理 | `claude mcp add sequential-thinking --scope user -- cmd /c npx -y @modelcontextprotocol/server-sequential-thinking` |
| `memory` | ✅ | 轻量知识图谱记忆（与 icm 重叠，可二选一） | `claude mcp add memory --scope user -- cmd /c npx -y @modelcontextprotocol/server-memory` |
| `context7` | ✅ | 库/框架最新官方文档，写代码前查版本行为 | `claude mcp add context7 --scope user -- cmd /c npx -y @upstash/context7-mcp` |
| `playwright` | ✅ | 浏览器自动化，真实用户路径验证（点击/输入/截图） | `claude mcp add playwright --scope user -- cmd /c npx -y @playwright/mcp` |
| `code-review-graph` | ❌ 故障 | 审查知识图谱 | 待修复 Python 环境 |

---

## 五、Agent 自动接入方式

让工具从「手动命令」变成「Agent 自动用」，三条路（按可靠性排序）：

1. **Hooks 全自动**（最可靠）：`icm init --mode all` 装的 hooks 会在会话开始自动注入记忆、会话结束自动归档——Agent 想忘都忘不掉。
2. **MCP 工具**（自动发现）：graft / codegraph 接入 MCP 后，Agent 能自动发现并按需调用 `graft_find_code`、`codegraph_context` 等。
3. **CLAUDE.md 行为规则**（次可靠）：在全局/项目 CLAUDE.md 里写明优先级，例如：

```markdown
- 结构性问题（谁调用谁、改 X 影响什么）→ 先用 codegraph / graft，不用 grep 盲搜
- 字面文本问题 → grep / read
- 跨会话需要记住的东西 → icm store；开工前 → icm recall
- 依赖版本行为不确定 → 先查 context7 再写代码
```

---

## 六、推荐组合

**最小可用组合（新机器 10 分钟配齐）**：

```powershell
npm install -g @nanonets/graft          # 代码图谱（首选）
irm https://raw.githubusercontent.com/rtk-ai/icm/main/install.ps1 | iex; icm init --mode all   # 记忆
claude mcp add context7 --scope user -- cmd /c npx -y @upstash/context7-mcp   # 文档
claude mcp add sequential-thinking --scope user -- cmd /c npx -y @modelcontextprotocol/server-sequential-thinking
```

**每个项目初始化**：

```powershell
graft init && graft build
codegraph init -i
```

**分工一句话**：graft/codegraph 让 Agent「看得懂仓库」，icm 让 Agent「记得住历史」，rtk 让 Agent「读得起输出」，context7 让 Agent「查得准文档」。四件套齐了，Claude Code 的上下文质量和检索速度会有质的提升。

---

## 更新维护

| 工具 | 更新命令 |
|------|---------|
| graft | `npm install -g @nanonets/graft`（或 `graft upgrade`） |
| codegraph | `npm install -g` 对应包 |
| icm | `icm upgrade` |
| rtk | 重新下载对应发布版 |
| MCP (npx 类) | 无需手动，npx 自动拉最新 |

## License

MIT
