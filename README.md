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

## 七、深度探索补充（v2，自动加载型，不含 skills）

> 筛选标准：**能被 Agent 自动识别/自动使用**（MCP、hooks、plugins），排除手动 slash-command/skills 类。

### 7.1 Serena MCP ⭐ 推荐（语义代码检索，免费）✅ 已接入

**仓库**：https://github.com/oraios/serena ｜ **状态**：✅ 已接入 user config，`claude mcp list` 验证 Connected

基于 LSP 的语义级 `find_symbol` / `find_referencing_symbols` / 符号级编辑，与 codegraph 互补（codegraph 偏图关系，Serena 偏精确符号操作）。

```powershell
claude mcp add serena --scope user -- cmd /c uvx --from git+https://github.com/oraios/serena serena start-mcp-server
```

**⚠️ 首装必读**：第一次连接会超时——uvx 需要从 GitHub 拉包构建（约 76 个依赖，1-3 分钟）。先手动预热一次缓存即可永久解决：

```powershell
uvx --from git+https://github.com/oraios/serena serena --version   # 预热，输出版本号即成功
claude mcp list                                                     # 再查就是 Connected
```

**自动使用机制（无需任何配置）**：
- MCP 工具对 Agent **自动可见**——会话里 Agent 看得到 `find_symbol`、`find_referencing_symbols`、`get_symbols_overview`、`replace_symbol_body` 等工具，按需自动调用
- Claude Code 会把任务语义与工具描述匹配：你说"重构这个函数"，Agent 自己决定先用 Serena 找符号/看引用再改——就像它自动用 Grep/Glob 一样，**不需要你在 prompt 里点名**
- 唯一注意：MCP 工具定义会占一点上下文（约 1-2k tokens），这是自动加载的代价

### 7.2 chrome-devtools-mcp ⭐ 推荐（真实浏览器调试）

**仓库**：https://github.com/ChromeDevTools/chrome-devtools-mcp

Google 官方。让 Agent 直接控制真实 Chrome：性能 trace、网络检查、console 读取、截图、性能分析——比 playwright 更贴近"前端调试"场景（playwright 偏自动化测试，这个偏调试）。

```powershell
claude mcp add chrome-devtools --scope user -- cmd /c npx -y chrome-devtools-mcp@latest
```

- **优点**：Google 官方维护；性能 trace / network 检查是 playwright 没有的；npx 自动更新。
- **缺点**：需要本机有 Chrome；和 playwright MCP 二选一按需开，都开会占上下文。

### 7.3 claude-context（Zilliz）— 暂不采用 ❌

语义向量检索全库。**不推荐进默认清单**：需要 embedding API（付费）+ 向量库，违背"全免费"原则。语义盲区由 graft/codegraph/Serena 三件套覆盖。若未来有免费 embedding 方案（本地 Ollama）可再评估。

### 7.4 beads (bd) — AI Agent 依赖感知任务追踪 ✅ 已装已接入

**仓库**：https://github.com/gastownhall/beads ｜ 本机已装：`bd version 1.1.0`

把任务/issue 建成**依赖图**持久化在 git 仓库里（Dolt 驱动），Agent 跨会话处理长任务不丢上下文，取代散乱的 markdown 计划。

**一次性接入（每个项目里执行）**：

```powershell
cd 你的项目
bd init               # 初始化 .beads/ + 本地 Dolt 数据库
bd setup claude       # 装 Claude Code 集成：SessionStart hook 自动注入任务上下文 + CLAUDE.md 规则块
```

**日常使用（Agent 自动走这套，你也可以手动）**：

```bash
bd create "实现登录接口" --type task --priority 1 -d "详细描述"   # 建任务
bd dep add <子任务id> <父任务id>    # 建依赖关系（形成 DAG）
bd ready                           # 关键命令：列出当前无阻塞、可立刻做的任务
bd update <id> --claim             # 认领任务
bd close <id>                      # 完成并关闭
bd show <id>                       # 看详情
bd list --status open              # 所有未完成任务
bd prime                           # 输出完整工作流上下文（给 Agent 看的说明书）
bd remember "踩坑经验"              # 项目级持久知识（替代 MEMORY.md）
bd stats                           # 统计
```

**自动加载机制**：`bd setup claude` 装的 SessionStart hook 会在每次会话开始时自动运行 `bd prime --hook-json`，把任务状态注入上下文——Agent 一睁眼就知道有哪些活、先干哪个。配合依赖图，Agent 永远不会跳步。

**同步机制**：任务数据在本地 Dolt DB，通过 git remote 的 `refs/dolt/data` 多机同步，`.beads/issues.jsonl` 是被动导出（人可读、可 review）。

- **优点**：`bd ready` 天然适配 Agent "下一步做什么"的决策；依赖图防止跳步；SessionStart hook 全自动注入，无需手动召回；数据随 git 走，可多人/多机协作。
- **缺点**：首次 `bd init` 拉起 Dolt 稍慢；别和 `workflow_status.md` 双轨记账，二选一。
- **验证状态**：✅ 已安装 1.1.0，`bd setup claude` 配方已验证；fork 项目贡献者模式 + 规划仓库全链路实测通过（create → ready → close）。

**全局一次性接入（任意目录跑一次，所有项目生效）**：

```powershell
bd setup claude   # 装全局 SessionStart hook（bd prime --hook-json）+ 全局 CLAUDE.md 规则块
```

**每项目激活（只需 `bd init` 这一步）**：

```powershell
cd 你的项目
bd init     # 初始化 .beads/（Dolt 嵌入库）。fork 项目会进贡献者向导：
            #   issues 自动路由到规划仓库（~/.beads-planning），不污染上游 PR
            #   向导后需在规划仓库补一次 bd init（Windows 版偶发漏建库）
```

**Hook 空转安全**：未 init 的项目里 SessionStart hook 静默输出空内容，零副作用——hooks 装一次全局生效，无需重复 `bd setup`。

### 7.5 Trail-Code（graft 的进化版，观察名单）

**仓库**：https://github.com/trailhq/Trail-Code

graft 作者的新项目：在代码图谱之上加了 **Learnings** 层（团队踩过的坑/决策自动钉在图谱上）。graft 用户关注，等 Windows 稳定后可替换。

### 7.6 已装但值得启用的（本机现状备注）

| 工具 | 状态 | 建议 |
|------|------|------|
| `context7` | ✅ 已升为全局 user 作用域 | 7.10 已验证 Connected |
| `bd` (beads) | ✅ 已装 1.1.0，`bd setup claude` 配方已验证 | 每个新项目：`bd init && bd setup claude`（详见 7.4 节） |
| `serena` MCP | ✅ 已接入全局（uvx 版，首装预热约 1-2 分钟，之后秒连） | 会话里直接用其 find_symbol 等工具。**启动时会自动弹 Serena Dashboard 浏览器页**（127.0.0.1:24282，查看配置/日志/项目记忆用），在弹出的页面点 **"Disable automatic opening"** 可永久关掉；点 "Open this whenever Serena starts" 则每次都开。无官方汉化，页面纯英文 |
| `tokentracker-cli` / `ccline` | 已装 | ccline 可做 statusLine 状态栏（显示上下文用量，防爆栈） |
| `code-review-graph` | ✅ **已修复**（2026-02）：旧 uv venv 被删导致 `failed to locate pyvenv.cfg`，`uv tool install --force code-review-graph` 重装后 Connected | 更新：`uv tool upgrade code-review-graph` |

### 7.8 全量扫描补充发现（本机已有、零安装）

对 npm 全局、`.local/bin`、choco、uv/bun 全量扫描后，已装但之前没列的：

| 工具 | 说明 | 建议 |
|------|------|------|
| `chrome-devtools-mcp@0.9.0` | npm 全局已装但**未接入 MCP 配置** | `claude mcp add chrome-devtools --scope user -- cmd /c npx -y chrome-devtools-mcp@latest` 即可激活（见 7.2） |
| `@upstash/context7-mcp@4.4.4` | npm 已装，仅项目作用域 | 升全局（见 7.6 表） |
| `omo-*`（Oh-My-Opossum 套件） | `.local/bin` 一整套：omo-lsp / omo-codegraph / omo-git-bash-hook / omo-ultrawork 等 | 已自装 hooks 套件，与现架构并存；如异常检查其 hooks |
| `ralph` (Ralph Orchestrator) | 自主循环编排器（npm @ralph-orchestrator/ralph-cli） | 适合"睡前挂机跑任务"场景，按需用 |
| `specify` | spec-kit 规格驱动开发 CLI | 按需 |
| `goose` | Block 的 AI agent CLI | 备用 agent，非 Claude Code 生态必需 |
| `bun@1.3.2` / `uv` | 运行时 | 已在为其他工具服务 |
| choco: ripgrep/ffmpeg/redis/nasm | 系统级依赖 | 无需动 |

**结论**：免费工具链已覆盖——图谱（graft+codegraph+Serena）、记忆（icm）、任务（beads）、输出压缩（rtk）、文档（context7 ✅）、调试（chrome-devtools ✅）、推理（sequential-thinking）。第二轮扫描新发现见 7.10。

### 7.9 观察名单（好但不满足"自动加载"或暂不成熟）
- **everything-claude-code (ecc)**：hooks 套件（agentshield 等），本机 CLAUDE.md 已引用其方法，可按需装。
- **claude-md-doctor**：CLAUDE.md 体检工具，一次性诊断用，非常驻。

### 7.10 第二轮全量扫描新发现（2026-02）

扫描范围：npm 全局清单复核、`.local/bin`/`.bun/bin`/choco/uv 复扫、D:\参考项目 各分类目录翻新、awesome-claude-code 与 claude-plugins-official 上游生态。

#### 已接入 ✅（实测 `claude mcp list` Connected）

| 工具 | 类型 | 接入命令（PowerShell） | 优点 | 缺点 | 验证 |
|------|------|----------------------|------|------|------|
| **context7** (Upstash) | MCP 文档检索 | `claude mcp add context7 --scope user -- cmd /c npx -y @upstash/context7-mcp` | 免费拉取库官方文档，版本敏感编码必备；Agent 自动调用 | 首次 npx 拉包需预热 | ✅ Connected |
| **chrome-devtools-mcp** | MCP 浏览器调试 | `claude mcp add chrome-devtools --scope user -- cmd /c npx -y chrome-devtools-mcp@latest` | Google 官方，干净 Chrome 实例做真实路径验证/性能追踪 | 每次新实例无登录态 | ✅ Connected |

> **Git Bash 注意**：`claude mcp add ... -- cmd /c npx ...` 在 Git Bash 里 `/c` 会被转义成 `C:/` 路径导致配置损坏——**必须用 PowerShell 执行 add 命令**。

同时清理了 graft 的 user/project 作用域重复定义（保留 user 作用域），`claude mcp list` 诊断区已无警告。当前 8/9 MCP Connected（仅 code-review-graph 故障，见 7.6）。

#### 新发现候选（实测可用，按需接入）

| 工具 | 类型 | 安装/接入 | 优点 | 缺点 | 与现有重叠 |
|------|------|----------|------|------|-----------|
| **Windows-MCP** (CursorTouch) | MCP，Windows 桌面 UI 自动化 | `claude mcp add windows-mcp --scope user -- cmd /c uvx windows-mcp` | 免费本地；打开应用/窗口控制/键鼠模拟，Claude Desktop 2M+ 用户 | Python 3.13+；UI 自动化有误操作风险，建议按需开启 | 互补（现工具链无桌面控制能力） |
| **mcp-chrome** (hangwin) | MCP，Chrome 插件式浏览器控制 | `npm install -g mcp-chrome-bridge` + 装配套 Chrome 扩展 | 复用**真实浏览器登录态**，20+ 工具（截图/网络监控/跨标签页） | 早期阶段；需手动装扩展 | 与 chrome-devtools-mcp 互补：后者干净实例调试，前者带登录态真实操作 |
| **claude-hud** (jarrodwatts) | statusline 插件 | Claude Code 内：`/plugin marketplace add jarrodwatts/claude-hud` → `/plugin install claude-hud` → `/claude-hud:setup` | 实时上下文用量条（>85% 红色预警，防 context 爆栈）、工具/子代理状态 | 插件命令交互式安装，未在本会话实测；与 ccline 功能重叠 | 与已装 ccline 二选一即可 |

#### Windows-MCP ✅ 已接入（实测 Connected）

**接入命令（PowerShell 执行）**：
```powershell
claude mcp add windows-mcp --scope user -- cmd /c uvx windows-mcp serve
```
⚠️ 注意两处坑：① 必须用 PowerShell（Git Bash 会把 `/c` 转义成 `C:/` 损坏配置）；② 新版必须有 `serve` 子命令，缺了会 CONNECTION_CLOSED。

**运行机制**：uvx 按需拉取 PyPI `windows-mcp`，本地运行零 API key，Agent 自动调用工具（MCP stdio）。

**19 个工具（来自源码 tools/ 目录）**：

| 类别 | 工具 | 能力 |
|------|------|------|
| UI 感知 | `Snapshot` / `DisplayInventory` | 抓取屏幕 UI 元素树（带 label，供其他工具定位）、显示器信息 |
| 键鼠操作 | `Click` / `Type` / `Scroll` / `Move` / `Shortcut` / `MultiSelect` / `MultiEdit` | 点击、输入（可按 label 定位）、滚动、快捷键（"ctrl+c"/"alt+tab"）、批量选择/多字段填写 |
| 窗口/应用 | `App` / `WaitFor` / `Wait` | 打开/切换应用、等待窗口出现、延时 |
| 系统 | `PowerShell` / `Process` / `Registry` / `FileSystem` | 执行 PowerShell、列出/杀进程、读写注册表（HKCU/HKLM）、文件操作 |
| 其他 | `Clipboard` / `Notification` / `Scrape` | 剪贴板读写、Windows toast 通知、网页抓取 |

**典型用途**：QA 测试桌面应用、自动化 GUI 操作（如安装向导）、配合浏览器工具覆盖非 Web 场景。

**风险与建议**：
- 键鼠模拟会**抢占真实鼠标键盘焦点**，运行期间不要动电脑；`PowerShell`/`Registry`/`Process kill` 工具有系统级破坏力
- 建议只在可信任务里用；若担心误操作，可临时 `claude mcp remove windows-mcp -s user` 卸载，或用其 `--config` 做工具白名单（新版支持 config.toml）
- 开机自启（可选）：`uvx windows-mcp install` 会注册计划任务（本机未启用，按需）

#### 仅对比说明（与已装重叠，不接入）

- **codebase-memory-mcp** (DeusData)：纯 C 后端、17 工具、162 语言、Hybrid LSP 类型解析，还有 3D 图谱可视化（CBM Lens）。功能上是 graft+codegraph+Serena 的超集候选，但三件套已稳定在用，迁移收益不明确 → 观察名单。
- **MCPProxy (mcpproxy-go)**：Go 单二进制 MCP 聚合代理——BM25 工具检索省 token、新 server 自动隔离防工具投毒、敏感数据检测。当前本机 MCP 仅 9 个，聚合收益低；等 server 数量上来再考虑。
- **claude-plugins-official 的 LSP 插件**（typescript-lsp / pyright-lsp / gopls-lsp 等）：官方插件把 LSP 诊断接入 Agent 自动使用，免费且零边际。按项目语言选择性启用，Windows 上启用方式：`/plugin marketplace add anthropics/claude-plugins-official`。

**结论更新**：context7 与 chrome-devtools 两个"待接入"已全部落地验证，免费工具链闭环完成。剩余高价值新方向只有 Windows 桌面控制（Windows-MCP）和带登录态浏览器（mcp-chrome），均按需启用。

### 7.11 第三轮全量扫描（2026-02，Windows-MCP 落地轮）

**修复记录**：code-review-graph 故障根因是原 uv venv 目录被删（exe 是 uv trampoline，启动时找不到 pyvenv.cfg），`uv tool install --force code-review-graph`（v2.3.9）重装即愈。Windows-MCP 已正式接入（见 7.10）。当前 **10/10 MCP 全部 Connected**。

**新发现候选**：

| 工具 | 类型 | 接入方式 | 优点 | 缺点/重叠 | 状态 |
|------|------|---------|------|----------|------|
| **context-mode** (mksglu) | MCP + 全套 hooks 插件（PreToolUse/PostToolUse/PreCompact/SessionStart） | `claude mcp add context-mode --scope user -- cmd /c npx -y context-mode`（完整版走 plugin 安装以启用 hooks 自动路由） | Hacker News #1；MCP 工具输出（Playwright 快照 56KB/次等）落盘索引、按需检索回填，直击 context 爆栈痛点；FTS5 全文索引本地免费 | 与 rtk 定位不同（rtk 压缩命令输出，context-mode 压缩 MCP 工具输出），互补；hooks 版本安装较重 | 待验证（未接入） |
| **qmd** (tobi) | 本地搜索引擎 CLI/MCP | npm/二进制，BM25+向量+LLM 重排全本地（node-llama-cpp GGUF） | 笔记/文档/转录本地语义检索，agentic flow 友好；完全免费离线 | 首次下载 GGUF 模型较大；与 graft/codegraph 定位不同（文档 vs 代码） | 待验证（未接入） |
| **Litho / deepwiki-rs** (sopaco) | Rust CLI，DeepWiki 类仓库 Wiki 生成 | `cargo install deepwiki-rs` 或 crates.io | 一键生成高质量 repo-wiki（中文文档全）；本地运行 | 需 Rust 工具链；生成型工具非常驻，按项目需要跑 | 按需（非常驻） |
| **claude-devtools** (matt1398) | 桌面可观测性 app | 按 README 安装（分析 Claude Code session 日志） | 回合级 context 数据、压缩可视化、子代理执行树 | 观测工具，非常驻，不进 MCP | 观察名单 |
| **Windows-MCP 官方 `install` 命令** | 开机常驻 | `uvx windows-mcp install`（计划任务）+ `auth` 生成密钥 | 常驻后免 uvx 冷启动 | 本机暂不需要 | 未启用 |

**扫描覆盖**：npm 全局 90+ 包复核、`.local/bin`/`.bun/bin`、choco、uv、D:\参考项目 全部 13 个分类目录（含 Agent 记忆/多智能体编排/文档处理/搜索增强/桌面系统工具），上游 awesome-claude-code 与 claude-plugins-official 复核。

**结论**：与现有工具链功能重叠外的空白已基本扫尽。下一个真正值得试的是 **context-mode**（与 rtk 互补的另一半 context 治理），其次是 qmd（本地文档语义检索）。

### 7.12 第四轮：context-mode 与 qmd 落地 + MCP 全景使用手册（2026-02）

#### context-mode ✅ 已接入完整版（MCP + 全套 hooks，实测 PASS）

```powershell
npm install -g context-mode                                          # 全局装（hooks 走稳定路径，不用 npx 冷启动）
claude mcp add context-mode --scope user -- cmd /c context-mode      # MCP
```

**hooks 完整版**（官方推荐走 `/plugin marketplace add mksglu/context-mode` 交互安装；本机直接把 4 个 hook 写进 `~/.claude/settings.json`，效果等同且免交互，Windows 兼容已验证——hooks 是纯 node .mjs，无 bash/jq 依赖，`context-mode doctor` 全 PASS）：
- **PreToolUse**（Bash/WebFetch/Read/Grep）：拦截大输出命令，改走沙箱索引
- **PostToolUse**：捕获工具输出自动落盘 FTS5 索引 + 会话记忆
- **PreCompact**：压缩前快照会话状态（决策/进行中任务不丢）
- **SessionStart**：注入 `<context_window_protection>` 路由指令（Think-in-Code 原则）

**核心工具**：`ctx_batch_execute`（一次跑多条命令+索引+检索）、`ctx_search`（含时间线式会话记忆查询）、`ctx_execute`/`ctx_execute_file`（沙箱跑代码只返回结论）、`ctx_fetch_and_index`（替代 WebFetch）。诊断：`context-mode doctor`。

**与 rtk 的分工**：rtk 压缩 Bash 命令输出（RTK.md 指令层），context-mode 压缩 MCP 工具输出 + 大文件读取 + 网页抓取（hooks+MCP 层）——互补共存。

#### qmd ✅ 已接入（实测 Connected，2.8.3）

```powershell
npm install -g @tobilu/qmd          # 安装 CLI
claude mcp add qmd --scope user -- cmd /c qmd mcp   # MCP（stdio）
```

**首次使用（每台机器一次性配置）**：
```powershell
qmd collection add "D:\我的笔记" --name notes   # 注册要索引的目录（默认 **/*.md）
qmd update                                      # 建索引（BM25 全文立即可用）
```
**检索档位决策**：只用 **BM25 档**（`qmd search "关键词"`，零模型零算力，毫秒级）。向量/混合档需要 `qmd pull` 下载本地 GGUF 模型 + `qmd embed` 全量向量化，**吃本机 CPU/GPU 算力，明确不启用**（语义检索需求由 graft/Serena 覆盖代码侧）。

**Agent 自动使用方式**：MCP 接入后 Agent 可直接调 qmd 工具检索已索引集合；CLI 也能让 Agent 跑 `qmd search`。定位与 graft/codegraph/Serena 互补——它们搜**代码结构**，qmd 搜**文档内容**（笔记/规格/README/转录）。

**本机已索引集合**：`rules`（Desktop\编码规则）、`agentcode`（本仓库）。

#### 附：MCP 全景使用手册（12 个 server，谁自动、谁要手动）

| # | MCP | 管什么 | Agent 是否自动用 | 需要手动做什么 |
|---|-----|--------|-----------------|---------------|
| 1 | serena | 语义代码检索（find_symbol 等），大文件精准定位 | ✅ 自动（工具就位，Agent 按需调用） | 无。首装预热 1-2 分钟已完成 |
| 2 | memory | 知识图谱长期记忆（跨会话实体/关系） | ✅ 自动 | 无 |
| 3 | sequential-thinking | 分步推理草稿纸 | ✅ 自动 | 无 |
| 4 | codegraph | 代码图谱查询（需项目先 `codegraph init -i`） | ✅ 自动（有图谱才用） | **新项目一次**：`codegraph init -i && codegraph index` |
| 5 | icm | 持久记忆主力（hooks 全自动存取） | ✅✅ 全自动（hooks 触发，无需提醒） | 无 |
| 6 | graft | 代码图谱+上下文检索 | ✅ 自动 | **新项目一次**：`graft init` |
| 7 | context7 | 拉取库官方最新文档（版本敏感编码前查） | ✅ 自动 | 无 |
| 8 | chrome-devtools | 干净 Chrome 实例：调试/截图/性能追踪/真实路径验证 | ✅ 自动（前端任务时 Agent 会调） | 无（会自动起浏览器） |
| 9 | windows-mcp | Windows 桌面 UI 自动化（19 工具，见 7.10） | ✅ 自动（工具就位） | 无。注意键鼠会抢焦点 |
| 10 | code-review-graph | 审查图谱（变更检测/影响半径） | ✅ 自动（有图谱才用） | **新项目一次**：`code-review-graph build` |
| 11 | qmd | 本地文档 BM25 检索（笔记/规格/README） | ✅ 自动（已索引的集合） | **每台机器一次**：collection add + update（**不接模型**，见 7.12）；新文档目录要 collection add |
| 12 | gitnexus | 架构级代码知识图谱（依赖链/调用链/盲改检测） | ✅ 自动（有索引才用） | **新项目一次**：`npx gitnexus analyze` |
| 13 | cocoindex-code | 语义代码搜索（自然语言查代码，Δ-only 增量） | ✅ 自动（已 init 项目） | **新项目一次**：`ccc init && ccc index`（未 init 项目 MCP 不可用） |

**总结**：全部 14 个都是"Agent 自动调用"型，日常零手动。唯一的维护动作是**新项目**跑一次图谱/索引初始化（`graft init` + `npx gitnexus analyze` + `ccc init && ccc index`），以及**新文档目录**加入 qmd 索引。

### 7.13 第五轮全量扫描（2026-02，13 大类目录全翻）

**GitNexus ✅ 已接入（1.6.12，实测 Connected）**——第 12 个 MCP：

```powershell
npm install -g gitnexus                     # CLI
# 在项目根目录（一次性，建索引+注册 hooks+生成 AGENTS.md/CLAUDE.md 上下文）：
npx gitnexus analyze                        # 本仓库实测 23.6s，37 节点/35 边
claude mcp add gitnexus --scope user -- cmd /c gitnexus mcp   # MCP
```

- **定位**：本地代码知识图谱（图数据库 LadybugDB）+ 架构级问答——比 graft 多"依赖链/调用链/盲改检测"视角，Agent 由此不再漏依赖
- **免费本地**：嵌入用本地 ONNX 运行时（`gitnexus embeddings` 管理），向量检索有 exact-scan 模式兜底，不强制下载大模型
- **维护**：新项目 `npx gitnexus analyze`；全局仓库列表 `gitnexus list`；卸载 `gitnexus uninstall`
- 与 graft/codegraph/Serena 并列使用，不冲突
**第五轮评估但未接入**（逐个详细展开，含优缺点与"如果要用"教程）：

---

#### 5-1. headroom（headroomlabs）— token 压缩器 ⚠️ 暂不接入（观察名单）

**是什么**：在工具输出/日志/RAG 块/文件到达 LLM 之前做本地压缩（官方示例：10,144 token 日志压到 1,260 且保留 FATAL 行），支持**可逆压缩 CCR**（原文本地缓存，需要时取回）。官方宣称四种形态：Python/TS 库、独立 proxy、`headroom wrap claude` 包装器、MCP server（`headroom_compress`/`headroom_retrieve`/`headroom_stats`）。

| 优点 | 缺点 |
|------|------|
| 压缩在本地执行，内容不出机器 | **PyPI 版(0.2.7)名不副实**：装完只有 `max` CLI，无 MCP server 入口（README 宣传的 MCP 在 GitHub main 未发版），装了也调不了（本机已实测 uv tool install 验证） |
| 功能最全：库/proxy/wrap/MCP 四形态 + 跨 agent 共享记忆 + `headroom learn` 反思写入 CLAUDE.md | **wrap 模式强制交互式 first-run**：必须选 LLM provider（OpenAI/Anthropic/ollama/local GGUF），且压缩引擎要调 LLM——付费 key 或本地算力，两条都踩本机红线 |
| CCR 可逆设计比 rtk 的一刀切更安全 | 与 rtk（命令输出）+ context-mode（MCP 输出+hooks）功能重叠，无独特增量 |

**如果要用**（等 GitHub main 发版后）：
```powershell
uv tool install --from git+https://github.com/headroomlabs-ai/headroom headroom
headroom wrap claude        # 交互式选 provider（ollama 免费但吃算力；API 则付费）
headroom unwrap claude      # 卸载
```
**触发改判条件**：官方发布带 MCP server 的正式版，且提供"不调 LLM 的纯规则压缩"模式。

---

#### 5-2. HexStrike AI（0x4m4）— 渗透测试 MCP ❌ 不接入（按需待取）

**是什么**：AI 驱动的网络安全自动化 MCP 平台，150+ 安全工具（nmap/nuclei/sqlmap/hydra/metasploit 等）+ 12 个自主攻击 agent（recon/exploit/bruteforce 专职），v6.0，MIT 开源。

| 优点 | 缺点 |
|------|------|
| 免费 MIT，标准 MCP 协议，Claude Code 直连 | **高风险工具集**：含主动攻击/exploit/暴力破解，误触发即违法或破坏系统 |
| 把 150 个 CLI 安全工具统一成 MCP 接口，侦察/漏扫效率高 | 安装重：git clone + python venv + requirements.txt + 底层还要装 nmap 等工具链 |
| 已进 Claude Desktop/Cursor 官方目录，社区活跃 | 浏览器 agent 需额外 browser 依赖；部分 agent 要 LLM key |

**如果要用**（隔离环境）：
```powershell
git clone https://github.com/0x4m4/hexstrike-ai.git D:	ools\hexstrike-ai; cd D:	ools\hexstrike-ai
python -m venv hexstrike-env; .\hexstrike-env\Scripts\Activate.ps1
pip install -r requirements.txt
python hexstrike_server.py   # 起 API 服务（默认 127.0.0.1:8888），再按仓库 docs 注册 MCP 客户端入口
```
**触发改判条件**：做授权渗透测试/CTF 项目时，在隔离 VM 里临时接入。

---

#### 5-3. cocoindex（CocoIndex-code）— AST 语义代码索引 MCP ✅ **已翻案接入**（v0.5.1，实测 Connected）

**是什么**：Rust 引擎的增量语义代码索引。`ccc` CLI + MCP：AST 感知分块、Δ-only 增量（只重处理变更文件）、本地嵌入（默认 Snowflake/snowflake-arctic-embed-xs，**仅 ~30MB 小模型，CPU 秒级**，不吃算力）、语义搜索按含义不按 grep。`ccc index` 还自带 hooks（SessionStart + PostToolUse 编辑后自动增量索引）和 `ccc grep` 结构化 grep（免索引）。

**本机实测数据**（Agent-Code 仓库）：
- 首次索引：**2.0 秒**（76 chunks / 4 files，含嵌入）
- 改动后重索引：**0.5 秒**（Δ-only 生效， graft 全量重建的对比优势坐实）
- 语义搜索：中文查询"增量索引去重缓存命中"命中 0.896 分相关段落 ✅

| 优点 | 缺点 |
|------|------|
| 增量真功夫：2s → 0.5s 实测；嵌入模型仅 30MB 无算力压力 | **MCP 只在已 `ccc init` 的项目里可用**（未 init 项目会 CONNECTION_CLOSED，算已知特性非故障） |
| `ccc grep` 结构化 grep 免索引即可用 | call graph/blast-radius 弱于 GitNexus——它是**搜索型**不是**图谱型** |
| 单文件嵌入式存储（SQLite），零数据库部署 | 与 Serena 语义搜索部分重叠（但 Serena 是 LSP 符号级，ccc 是自然语言级，互补） |

**接入命令（已执行）**：
```powershell
uv tool install 'cocoindex-code[full]'      # 含本地嵌入（sentence-transformers）
# 首次免交互：预写全局设置（或跑 ccc init 交互）
# ~/.cocoindex_code/global_settings.yml:
#   embedding:
#     provider: sentence-transformers
#     model: Snowflake/snowflake-arctic-embed-xs
# 每个项目根目录：
ccc init          # 交互一次（或直接 ccc index 自动以默认设置初始化）
ccc index         # 建索引（2s 级）
claude mcp add cocoindex-code --scope user -- cmd /c ccc mcp
```
日常：编辑后 hooks 自动增量索引（需装其 hooks）；手动 `ccc index` 也只要 0.5s。搜索 Agent 自动调用（MCP `search` 工具，refresh_index 默认 true）。

#### 5-3.1 cocoindex-code 完整使用教程（本机已配置）

**三种角色**：`ccc` CLI（你/Agent 在终端跑）、MCP server（Claude Code 自动调用）、hooks（编辑后自动索引，可选装）。

**一、新项目接入（每次 30 秒）**
```powershell
cd C:\path\to\新项目
ccc init                 # 交互式：一路回车（默认 snowflake-arctic-embed-xs）
                         # 生成 .cocoindex_code/（settings.yml + SQLite 索引，加进 .gitignore）
ccc index                # 建索引（小仓库 2s，大仓库按体积）
```
之后 MCP 才在这个项目可用（MCP 进程的 cwd=项目根，`ccc mcp` 从 `.cocoindex_code/` 读索引）。

**二、日常使用（谁是主角）**
| 场景 | 谁来做 | 命令/行为 |
|------|--------|----------|
| 代码改了，索引旧了 | ① 手动：终端/会话里让 Agent 跑 `ccc index`（0.5s）② 自动：装其 hooks（见下） | `ccc index` |
| 想语义搜代码 | **会话里直接用自然语言问 Agent**："项目里哪里处理了重试逻辑？" → Agent 自动调 MCP `search` 工具 | 零命令 |
| 想按结构精确搜（函数名/正则） | 不用 ccc——用 graft/Serena/普通 grep | — |
| 想结构化 grep（给一段示例代码找相似代码，免索引） | 终端或 Agent 跑 | `ccc grep "def foo(self, x): return x*2"` |
| 看索引健康 | `ccc status`（项目级）/ `ccc doctor`（全局体检）/ `ccc daemon` 状态 | — |
| 索引坏了重来 | `ccc reset`（项目级，`--all` 连设置一起清） | — |

**三、可选：编辑后自动增量索引（hooks）**
仓库的 `hooks/hooks.json` 提供两条：SessionStart + PostToolUse（Edit/Write 触发 `ccc index`，仅当项目存在 `.cocoindex_code/settings.yml` 时生效，安全空转）。想要全自动就把这两条 hook 加进 `~/.claude/settings.json`（写法同 context-mode 接入方式，command 为 `ccc index`）。**本机暂未启用**——项目少时手动 `ccc index` 更可控，等项目多了再开。

**四、什么时候用 ccc vs 其他（速查）**
| 你要找… | 用 |
|---------|-----|
| "处理重试逻辑的代码在哪"（含义级） | **ccc**（语义） |
| "谁调用了这个函数 / 改这行影响什么"（结构级） | GitNexus / graft（图谱） |
| "这个符号的定义/引用/重命名"（LSP 精确级） | Serena |
| "README/笔记/规格里写过什么"（文档） | qmd |

**五、更新与卸载**
```powershell
uv tool upgrade cocoindex-code   # 升级
uv tool uninstall cocoindex-code # 卸载（项目里 .cocoindex_code/ 目录手动删）
```

**与图谱三件套分工**：graft/GitNexus 管**结构**（依赖链/影响半径），ccc 管**含义**（"哪段代码处理重试"这种 grep 不出来的问题），Serena 管**符号**（LSP 精确跳转）。四者并存。

---

#### 5-4. graphify — 代码库知识图谱可视化 ❌ 不接入（skill 依赖 + 重叠）

**是什么**：把代码库映射成交互式知识图谱 `graph.html`（力导向图，节点=概念，颜色=社区检测，可点击/过滤/查路径），终端还能查最短依赖路径并逐跳高亮。CLI 安装时自动注册 **skill** 到 `.claude/skills/graphify/`。

| 优点 | 缺点 |
|------|------|
| 可视化漂亮，架构汇报/新人 onboarding 有用 | **核心走 skill 注册**（`/graphify .` 手动触发）——skill 类无法自动加载，已排除 |
| 一次安装 20+ agent 通用，Windows 支持 | 图谱数据与 graft/GitNexus 重复建设 |
| 纯本地免费 | 依赖 Python 3.10+ 与 uv/pipx |

**如果要用**（当一次性分析工具，不常驻）：
```powershell
uv tool install graphifyy
cd <项目>; graphify install --project   # 写入项目级 skill
# 会话里 /graphify . 生成 graph.html，浏览器打开
```

---

#### 5-5. agent-deck — 多 AI 会话任务控制台 ❌ 不接入（Windows 需 WSL，硬伤）

**是什么**：终端 TUI 的"AI agent 任务控制台"，构建在 **tmux** 之上——一个终端管理所有 Claude Code/Codex 会话。

**全貌（功能清单）**：
- **会话总览**：tmux 状态栏显示等待中的会话，`Ctrl+b` + 数字 1-6 直接跳转；智能状态检测（区分 Claude 在思考 vs 在等待输入 vs 出错）
- **MCP Manager**：按 `m` 打开，`Space` 开关，Tab 切换作用域（项目/全局）——**不改 json 就能按会话启停 MCP**
- **MCP Socket Pool**：多会话通过 Unix socket 共享 MCP 进程（`pool_all = true`），宣称省 85-90% 内存，MCP 崩溃 3 秒自动重连
- **Skills Manager**：按 `s` 管理技能池（`~/.agent-deck/skills/pool`），按项目挂载/卸载
- **Conductor 指挥层**：起多个"指挥官"会话管理子 agent 会话（`agent-deck conductor setup ops`），带状态迁移通知守护进程（子会话 running→waiting/error 时提醒父会话）；可接 **Telegram/Slack bridge** 手机远程监控发指令
- **会话 fork**：带上下文继承地复制会话
- **全局搜索**：跨会话搜对话内容

| 优点 | 缺点 |
|------|------|
| MCP Socket Pool + MCP 开关 UI 是真价值（13 个 MCP 多会话并发时内存显著省） | **原生不支持 Windows**：官方 FAQ 明确要求 **WSL2 + tmux**（MCP socket pooling 依赖 Unix socket），Windows 原生 Claude Code 会话用不了 |
| Conductor/Telegram bridge 对"挂机多 agent"场景很强 | 本机已有 windows-mcp/omo hooks 生态，WSL 里再养一套运维成本高 |
| 会话 fork/全局搜索是 tmux 没有的 AI 增强 | 学习曲线：conductor/profile/bridge 三层概念 |

**如果要用**（WSL 环境）：
```bash
# WSL 内执行（非 PowerShell）
curl -fsSL https://raw.githubusercontent.com/asheshgoplani/agent-deck/main/install.sh | bash
# 或 brew install asheshgoplani/tap/agent-deck
agent-deck                    # 启动 TUI
# ~/.agent-deck/config.toml 声明 MCP 并开 pool_all = true
agent-deck conductor setup ops   # 起指挥官会话
agent-deck uninstall --keep-data # 卸载（保数据）
```
**触发改判条件**：① 日常 3+ 并发会话（池化收益）且愿意进 WSL；或 ② 需要 Telegram/Slack 手机远程盯 agent 时。

---

#### 5-6. token-savior（4.24.0）— 会话记忆/压缩 MCP ❌ 不接入（算力 + 重叠）

**是什么**：会话级记忆档案 MCP，宣称 tsbench 97.9% 准确率同时省 80% token。核心 `ts_search` 对历史会话观察做检索；向量档用 embedding warm-up；`TS_AUTO_EXTRACT=1` 用 LLM 自动提取记忆（需 Anthropic API key）。

| 优点 | 缺点 |
|------|------|
| BM25 档免费可用，pip 单装简单 | **向量档要 embedding warm-up（算力红线）**；auto-extract 要付费 key（付费红线） |
| "记忆观察"设计比纯文本压缩更结构化 | 与 icm（hooks 全自动记忆）+ context-mode（会话快照+timeline 检索）完全重叠 |
| 官方有基准数据背书 | 本机已有两套更强记忆接入，第三套无增量 |

**如果要用**：
```powershell
pip install "token-savior-recall[mcp]"   # 仅 BM25 档；设 TOKEN_SAVIOR_NO_WARMUP=1 跳过 embedding 预热
```
| graft | `npm install -g @nanonets/graft`（或 `graft upgrade`） |
| codegraph | `npm install -g` 对应包 |
| icm | `icm upgrade` |
| rtk | 重新下载对应发布版 |
| MCP (npx 类) | 无需手动，npx 自动拉最新 |
| windows-mcp | uvx 自动拉最新，无需手动 |

## License

MIT
