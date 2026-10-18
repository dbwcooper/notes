# DSH 桌面端升级到 0.2.1-rc.2 — 实测记录（2026-10-18）

> 来源：DSH 发布简报 `2026-10-18`（桌面端 nightly 高于本机已装）。本文是把该简报的升级待办做完后的实测结论，逐条对应待办 spec 的 8 项验证清单。

## 结论先说

- 本机已在 **0.2.1-rc.2**：`Info.plist`（`CFBundleShortVersionString` / `CFBundleVersion`）、`dsh --version`、`app.asar` 内 `@deepseek-ai/dsh-desktop-runtime` 三处一致。
- **升级由 App 内置自动更新完成，不是手工替换**：Squirrel 于 **2026-10-18 09:26:26–09:26:31** 安装并重启应用（因此 09:00–09:02 生成的简报里「本机 0.2.0-rc.2」在当时是准确的，二者不矛盾）。
- 安装包完整性可核对：nightly feed 的 `sha512` == 官方 zip 实测 `sha512` == updater 暂存 `update-info.json` 里的 `sha512`；已装 `app.asar` 的 sha512 与 zip 内 `app.asar` **完全一致**。
- 8 项验证：**6 项通过**，2 项只能给出「未确认」（涉及 App 界面视觉效果，命令行侧无从取证）。**无回归**，未回滚。

## 升级事实与完整性

| 项目 | 实测值 |
|---|---|
| nightly feed | `version: 0.2.1-rc.2`，`releaseDate 2026-10-17T09:57:19.892Z`，`size 347777642` |
| 下载 zip | 347777642 B，`sha512` = `vgGV720lZGEdMn…9Ydw==`（base64），与 feed 一致 |
| 安装时间 | ShipIt stderr：`2026-10-18 09:26:26` 起，`09:26:31 Successfully launched application` |
| updater 暂存 | `~/Library/Caches/@deepseek-aidsh-desktop-updater/pending/deepseek-harness-0.2.1-rc.2-mac-arm64.zip`，`update-info.json` 的 sha512 同上 |
| `app.asar` | 已装与 zip 内 sha512 均为 `d6004801d18f2440…c97c2b6`（182833898 B） |
| 版本字符串 | `0.2.1-rc.2`（Info.plist 两处 / 运行时 package.json / CLI） |
| CLI | `~/.local/bin/dsh` 仍软链到 App bundle 内 runtime，随 App 自动跟随 |

> 待办 spec 的步骤 1（先备份 profile、把旧 App 改名保留）**已无法按原样执行**：自动更新已直接替换 App，旧 0.2.0-rc.2 bundle 未留副本（ShipIt 的临时目录已被清理）。

**备份（升级后补做）**：`~/.dsh/backups/upgrade-0.2.1-rc.2-20261018/` 存了 `{desktop,web,headless}` 的 `package.json` + `cordis.patch.yml`，以及 `storages/schedule.json`。

**回滚路径**：旧 App 无本地副本，但 `https://download.deepseek.com/dsh-desk/bin/mac-arm64/deepseek-harness-0.2.0-rc.2-mac-arm64.zip` 实测 `HTTP/2 200`（`last-modified 2026-09-29`），需要时可下载替换 App 并重启；该包不在 feed 里，无官方 sha512 可比对。

## 8 项验证清单

### 1. 版本核对 —— ✅ 通过

- `Info.plist`：`CFBundleShortVersionString = CFBundleVersion = 0.2.1-rc.2`
- `dsh --version`：`0.2.1-rc.2`
- `Resources/app.asar` → `/dsh/package.json`：`@deepseek-ai/dsh-desktop-runtime 0.2.1-rc.2`

### 2. schedule bundle 是否被自动清理 —— ✅ 通过（自动化任务保留）

- 升级前 6 个 bundle 中的 `@deepseek-ai/dsh-experimental-schedule-bundle` **已被移除**；`~/.dsh/profiles/desktop/package.json` 在 09:26 应用启动时被重写为 5 个：`dsh-base`、`dsh-web-app`、`dsh-experimental-voice-input-bundle`、`dsh-experimental-agent-team-profile`、`dsh-experimental-auto-review`。
- 机制确认（0.2.1-rc.2 的 `@deepseek-ai/dsh-app-boot`）：`RETIRED_BUNDLES = {"@deepseek-ai/dsh-experimental-schedule-bundle"}`，profile 加载时 `dropRetiredBundles()` 会把它剔除并回写 manifest —— 与实测文件变化吻合。
- 自动化任务数据未丢：`~/.dsh/storages/schedule.json` 仍保留 1 条 active 任务「Todos #1 自动完成巡检（每日 00:30）」，`lastDelivery 2026-10-18 00:30`（升级前最后一次投递）。自动化已改为 Web 内置能力，**升级后首次触发要等 10-19 00:30，属未确认**。
- 其余 5 个 bundle 在本机安装内均可解析：把它们连同用户 patch 复制到临时 DSH_HOME 跑 `dsh --dump-config`，组合出 316 条 entry、无 `skippedBundles`、无 schema 报错。

### 3. magpie 自定义提供方配置是否被改写 —— ✅ 通过（内容未变）

`~/.dsh/profiles/desktop/cordis.patch.yml` 实测仍为手写形态：注释行在、`# magpie` 标记在，说明不是 YAML 序列化器重写的结果。

- 两个模型条目都在，且自定义模型名 `deepseek/deepseek-v4-pro`（name `DeepSeek V4 Pro · DeepSeek`）保留；
- 两个模型的 `reasoningEfforts: {low: low, high: high, max: max}` 均保留；
- `agent-default-model` 仍是 `provider: deepseek-account / model: deepseek-flash / reasoningEffort: high`。

schema 兼容性：0.2.1-rc.2 的 `dsh-llm-pi-ai` 仍以 `reasoningEfforts` 为合法字段（`z.dict(...)`，README 亦保留该写法），`--dump-config` 用真实 patch 组合无报错 —— 即 alpha.4 的「推理等级 UI 化」没有作废这份手写配置。

### 4. headless 仍能取 key —— ✅ 通过

- 配置未变：`headless` profile 的 `llm-pi-ai` 仍用 `apiKeyEnv: MAGPIE_GATEWAY_KEY` + `baseURL http://127.0.0.1:3425/v1`；`~/.dsh/.env` 里 `MAGPIE_GATEWAY_KEY`、`TDS_API_KEY` 均在。
- 实跑（临时 cwd，避免污染工作区）：
  `dsh --profile headless "只回复两个字：正常。不要调用任何工具。"` → 输出 `正常`。
- 会话日志佐证真正走了网关而非回退：`"provider":"magpie"`、`"model":"deepseek/deepseek-flash"`。
- 结论：rc.1 的「headless 不再读已保存凭据、改用环境变量/.env」对本机无影响。

### 5. MCP `todos` 工具 —— ✅ 通过

- 直连 `https://todos.dev/api/mcp`（`TDS_API_KEY`）`initialize` 成功（`serverInfo: todos 0.1.0`），`tools/list` 返回 **31 个工具**（spec 里写的「3 个工具」是旧基线，现已扩到 31 个：`todos / projects / create_todo / … / schedules / machines / issues …`），无缺失。
- 升级后的 App 会话里 MCP 调用真跑通过：`session-4f60590e…` 在 **09:30** 调 `mcp__todos__todos` 两次，`isError=False`（一次返回 3 条待办、一次返回 #3 详情）；10:32 起的会话也有 `mcp__todos__*` 调用。
- 历史里 `mcp-client(todos): server is disconnected` 只出现在 10-11，与本次升级无关。

### 6. rc.2 两项能力 —— 部分确认

**(a) DeepSeek-V4.1-Flash-1017：未以独立模型条目出现，而是「官方 Flash 条目 + 1017 徽标」**

- 0.2.1-rc.2 的官方目录（`dsh-llm-deepseek`）只有 `deepseek-flash`（name `DeepSeek-V4.1-Flash`）与 `deepseek-v4-pro`，没有 `-1017` 的独立 id。
- 新增 `@deepseek-ai/dsh-client-ui-deepseek-model-label-1017`：在 `conversation.input.model.label` slot 上，仅当 `provider.id ∈ {deepseek-account, deepseek-official}` 且 `model.id === "deepseek-flash"` 且 `model.name === "DeepSeek-V4.1-Flash"` 时追加 `1017` 标签。
- 本机情况：桌面默认模型是 `deepseek-account / deepseek-flash`，若账号已登录即可命中徽标；而本地网关（magpie）的模型 id 是 `deepseek/deepseek-flash`、显示名 `DeepSeek V4.1 Flash · DeepSeek`，**不满足匹配条件，不会显示 1017**。徽标的实际视觉效果：**未确认**（未截图）。

**(b) DeepSeek Search / Fetch API：可开启，且与本地网关「叠加」已有两条可用路径**

- 内置挂载（`dsh-base`）：`web`（`searchProvider: deepseek-official`、`fetchProvider: http`）、`web-search-deepseek`（Messages 搜索，`apiKeyEnv: DEEPSEEK_API_KEY`，端点 `https://api.deepseek.com/anthropic/v1`）、`web-deepseek-api`（新版 Search/Fetch API，`https://api.deepseek.com/v1/tools`）；插件页「网页搜索」设置页由 `dsh-client-ui-settings-web-search` 提供 —— 因此「启用新版 Web 搜索 API」开关在界面上存在。
- 当前状态：桌面 patch 里**没有 `web` 段** → 仍是默认组合（Messages 搜索 + 本地 HTTP 抓取），新版 API **未开启**（本会话未替用户改开关）。
- 新版 API 实测可用（用 `~/.dsh/.credentials.yaml` 里已有的 `DEEPSEEK_API_KEY`）：
  - `POST https://api.deepseek.com/v1/tools/search` `{query,count:2,need_content:false}` → `HTTP 200`，返回 `results[]`（标题/URL/摘要/日期）；
  - `POST https://api.deepseek.com/v1/tools/fetch` `{url: <release page>}` → `HTTP 200`，返回该页 Markdown。
- 与本地网关叠加实测：`POST http://127.0.0.1:3425/v1/messages`，带 `tools:[{type:"web_search_20250305",…}]` → 正常返回 `server_tool_use` + `web_search_tool_result`（真实检索结果块）。即 Messages 搜索可以改走本地网关：把「Messages API 基础地址」设为 `http://127.0.0.1:3425/v1`、模型设为 **`deepseek/deepseek-flash`**（网关不接受 `deepseek-v4-flash`，会报 `not_found_error`）；新版 Search/Fetch API 则走 DeepSeek 官方端点、与聊天模型互不干扰，两者可同时用。

### 7. `@deepseek-ai/dsh-plugin-market` 三件套 —— ✅ 已落地

- 安装内三个包齐全：`dsh-plugin-market`（Host 服务）、`dsh-client-ui-plugin-market`（插件页「发现」标签）、`dsh-experimental-plugin-market-bundle`（可选组合包，静态 patch 插入前两者）。
- 该组合包是**可选**项（`OPTIONAL_BUNDLES` 列表），不在任何默认模板里；入口是 插件页 → 「实验性插件」→ 更多 → 启用「自定义插件市场」。
- 本机状态：`~/.dsh/profiles/desktop/package.json` 于 **14:05:28** 被 App 侧写入，bundles 增加了 `@deepseek-ai/dsh-experimental-plugin-market-bundle`（命令行 `dsh --profile desktop` 会直接报「managed exclusively by the Electron application」，已验证 CLI 不会写这个文件），即 App 内已经把该实验组合包启用，「发现」/索引源入口随之出现。视觉呈现本身：**未确认**。
- **决定：保留启用状态观察，但不添加任何第三方索引源。** 理由：目前没有官方插件市场（包 README 明说「发现标签页只列出用户添加的索引源」），索引源来自第三方、安装的是第三方 bundle，收益（本机没缺插件）小于引入未审核依赖的风险；有具体需求时在「发现」里加一条 URL 即可，无需再改配置。

### 8. 日常链路不回归 —— ✅ 通过（次日运行待观察）

- `launchctl print gui/501/com.cooper.dsh-release-digest`：`state = not running`、`runs = 10`、`last exit code = 0`；plist 仍为 09:00 `StartCalendarInterval` + `monitor-dsh.sh --once`。
- 今晨 09:00 采集正常：`launchd.out.log` 末段记到 `[monitor] 4 项变更 @ 2026-10-18T01:00:11Z`，`changes.log`/`releases/` 均已更新。
- 简报任务继续可用：`2026-10-18.md` 已生成；本轮待办本身即由该简报派生并跑通。
- `mcp-todos` 正常（见第 5 项）。升级发生在 09:26，因此「次日 09:00 采集」与「自动化任务 10-19 00:30 首次触发」都要等到明天才能确认 —— **未确认**。

## 未确认项（不作「已是最新/一切正常」的断言）

1. 「1017」徽标在模型选择器里的实际渲染（代码条件已确认，未截图）。
2. 「网页搜索」设置页里开启新版 API 后保存、生效的界面流程（API 与凭据已实测可用，未在 UI 点击保存）。
3. 插件页「发现」标签与索引源入口的视觉呈现。
4. 自动化任务页是否按 rc.1 所述展示为 Web 内置能力；10-19 00:30 的首次触发。
5. launchd 次日 09:00 的采集（本次只能证明今天 09:00 成功）。

## 建议

- 若想让聊天模型也显示 `1017` 徽标，需使用官方 `deepseek-account` 提供方的 Flash 条目；走本地网关的 magpie 条目不参与该徽标。
- 想省 DeepSeek 官方搜索额度又保留网页搜索能力，可把「Messages API 基础地址」指向本地网关并把模型改成 `deepseek/deepseek-flash`（已验证网关支持 `web_search_20250305`）；想用新版 Search/Fetch API 则保持官方端点 + 已有 `DEEPSEEK_API_KEY`。
- 回滚只在发现阻塞性问题时做：下载 0.2.0-rc.2 官方 zip 替换 `/Applications/DeepSeek Harness.app` 后重启，并用本目录备份还原三个 profile。
