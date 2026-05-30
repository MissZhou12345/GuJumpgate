# Project Context

## Purpose

GuJumpgate 是一个 **Chrome Manifest V3 浏览器扩展**，用于全自动执行多步骤 OAuth 注册与 ChatGPT Plus 激活流程。

核心目标：

- 解放双手：串联 OpenAI 注册、邮箱/短信验证码、OAuth 同意、平台回调（CPA / SUB2API / Codex2API 等）及 Plus 付费（PayPal / GoPay）全流程。
- 可维护：以 `flow + workflow node` 模型组织步骤，避免回到硬编码步骤号驱动。
- 可扩展：通过 `data/step-definitions.js`、`background/steps/registry.js` 与 `shared/source-registry.js` 接入新邮箱来源、接码平台与注册模式。

阅读顺序（AI 开发前必读）：

1. [项目文件结构说明.md](../项目文件结构说明.md)
2. [项目完整链路说明.md](../项目完整链路说明.md)
3. [项目开发规范（AI协作）.md](../项目开发规范（AI协作）.md)

## Tech Stack

- **运行时**：Chrome Extension MV3（Service Worker `background.js` + Side Panel + Content Scripts）
- **语言**：原生 JavaScript（ES 模块风格通过 manifest 脚本列表装配，无打包器）
- **存储**：`chrome.storage.local`、DNR 规则（`rules.json`）
- **辅助服务**：
  - `services/checkout-converter/`：Python checkout 链接转换（Docker 可选）
  - `scripts/hotmail_helper.py` 等：本地 Hotmail / 账号快照 helper
- **接码**：`phone-sms/providers/`（HeroSMS、5sim、NexSMS、SMSPool 等）
- **规范与协作**：OpenSpec（`openspec/`）、Cursor/OpenCode 命令（`.cursor/commands/`、`.opencode/command/`）
- **包管理**：根目录 `package.json` 仅作 MIT 元数据；无前端构建链

## Project Conventions

### Code Style

- 使用语义化文件名，**禁止**新增 `stepX.js` 式步骤命名。
- 新增/修改函数时添加**函数级注释**（项目与用户规则要求）。
- 中文文案、日志、注释、提交信息使用自然中文；提交信息不得使用空泛的 `update` / `fix` / `AI 修改`。
- 乱码、错码、异常替换字符视为**阻塞问题**，必须当场修复。
- 修改过的 `.js` 文件提交前运行 `node --check <file>`。
- 工具函数放 `*-utils.js` 或 `background/*.js` 共享层；**不要**把新业务堆回 `background.js` 入口壳。

### Architecture Patterns

分层与职责：

| 层级 | 路径 | 职责 |
|------|------|------|
| 入口壳 | `background.js` | 装配、初始化、少量保留函数 |
| 步骤执行 | `background/steps/*.js` | 单步骤业务，经 `registry.js` 注册 |
| 步骤定义 | `data/step-definitions.js` | `FLOW_DEFINITION_BUILDERS`、`getSteps` / `getNodes` / `getWorkflow` |
| 流程能力 | `shared/flow-capabilities.js`、`flows/openai/` | flow 级能力与邮件规则 |
| 内容脚本 | `content/*.js` | 页面 DOM 自动化、验证码读取 |
| 侧栏 UI | `sidepanel/*.js` | 配置、状态展示、手动跳过/重试 |
| 纯工具 | 根目录 `*-utils.js` | 无 Chrome API 的归一化与解析 |

关键约定：

- **`activeFlowId`**：当前 flow（默认 `openai`）。
- **`step.id`**：legacy 可见步骤号，仅用于 UI 与人工沟通；执行与状态机以 **`key` / `nodeId`** 为准。
- **步骤定义单一来源**：改步骤必须同时更新 `step-definitions.js` 与 `registry.js`；sidepanel 不得维护另一套步骤列表。
- **注册邮箱状态**：`background/registration-email-state.js` 统一 `registrationEmailState`。
- **消息路由**：业务消息经 `background/message-router.js`，不直接在入口散落 handler。
- **OpenAI 手机号验证**：`background/phone-verification-flow.js` 属 OpenAI flow 边界，不作过早的 core 抽象。

### Testing Strategy

- 当前仓库**无**自动化 `npm test` 脚本；以 **手动串行跑通**（README：Chrome 无痕 + 代理环境）为主。
- 提交前：`node --check` 覆盖改动 JS；定向验证本次改动的步骤流、状态流、日志流、UI 流一致性。
- 阶段化开发：每阶段自检通过后再进入下一阶段（见 [项目开发规范（AI协作）.md](../项目开发规范（AI协作）.md) §0.3）。
- 涉及库/API 文档时优先查 Context7 MCP；涉及 OpenSpec 变更时运行 `openspec validate <change-id> --strict --no-interactive`。

### Git Workflow

- **日常开发基线**：`dev`（非 `master`）。PR 目标分支为 `dev`；发版/合并到 `master` 走单独发布流程。
- 发起 PR 前对齐 `origin/dev`；使用 `gh` CLI 操作 GitHub。
- 小步、清晰提交；不提交密钥、Cookie、代理、真实账号记录。
- 敏感本地方案默认放 `docs/md/`（gitignore），除非用户明确要求提交。
- OpenSpec：**先提案、评审通过后再实现**；部署后归档到 `openspec/changes/archive/`。

## Domain Context

- **主流程**：OpenAI 账号注册（邮箱或手机号）→ 验证码 → 资料 → OAuth → 可选 Plus（Stripe checkout → PayPal/GoPay）→ 平台 session 导入/回调验证。
- **邮箱来源**：QQ、163/126、Gmail、iCloud、Duck、Cloudflare Temp Email、Cloud Mail、2925、Hotmail/Outlook、自定义邮箱池等；各 provider 有独立 utils + content/background 模块。
- **Plus 模式**：步骤编号与普通模式不同，内部可复用节点（如登录验证码复用 Step 8 执行器但 UI 显示不同可见步）。
- **IP 代理**：PAC、`711proxy` provider、成功阈值后切换；与 PayPal/US 注册环境强相关。
- **多注册 flow**：架构已支持 `flowId`；新 flow 应沿 `FLOW_DEFINITION_BUILDERS` 与 `flows/<name>/` 扩展，而非复制整套 sidepanel/background 逻辑。

## Important Constraints

- Chrome MV3：Service Worker 无持久 DOM；标签与内容脚本通信经 `tab-runtime.js` 与消息协议。
- 权限敏感：`debugger`、`proxy`、`<all_urls>` 等；变更 manifest 需评估安全与商店策略。
- 公开仓库默认**不内置**第三方贡献 OAuth 服务端地址；需用户自行配置。
- 禁止在无明确需求时保留“保险式”旧分支兼容或职责泄漏的隐藏步骤。
- 用户说「只分析」时不得改代码；用户说「开始开发/提交」须按清单执行到完成。
- **实现门禁**：OpenSpec change 未批准前不开始编码。

## External Dependencies

- **OpenAI / Auth0**：`auth.openai.com`、`auth0.openai.com`、`accounts.openai.com`
- **支付**：`pay.openai.com`、`checkout.stripe.com`、`paypal.com`
- **邮件网页**：QQ、163/126、Gmail、iCloud、2925 等（content script 匹配域见 `manifest.json`）
- **接码平台 API**：HeroSMS、5sim、NexSMS、SMSPool、GrizzlySMS、SMSBower 等（`phone-sms/providers/`）
- **面板/回调**：CPA、SUB2API、Codex2API（`background/panel-bridge.js`、`sub2api-api.js` 等）
- **可选本地 helper**：Hotmail 收信、checkout 转换服务、GPC GoPay API（用户自配域名与 Key）
