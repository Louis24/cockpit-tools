# 🛩️ Cockpit Tools

> 本地运行的 AI IDE 账号管理器：多账号集中管理、一键切换、配额监控、多实例并行、每日签到 ✨

这是 [jlcodes99/cockpit-tools](https://github.com/jlcodes99/cockpit-tools) **v1.3.35 的个人分支**。功能已开发完结，仓库作为一份「能编译、能跑、踩过的坑都有记录」的完整实现保留在这里 🧾

| 项目状态            | 运行平台                  | 发布                         |
| ------------------- | ------------------------- | ---------------------------- |
| ✅ 已完结（2026-11） | 🪟 Windows 10 / 11（实测） | 📦 不提供二进制、不做自动更新 |

没有云端、不需要注册、不上传任何数据 🔒 所有凭据留在本机，用自己的密钥加密。

---

## 1. 😵‍💫 它解决什么问题

同一台电脑上往往有好几个 AI IDE 账号：公司的、自己的、试用的。官方客户端只记一个登录态，切号要退出再登录，还会遇到 refresh token 轮换导致的「越切越掉」、多开实例互相踢、以及根本看不见还剩多少额度 😮‍💨

Cockpit Tools 把这些账号统一收在一处：凭据加密存在本地，点一下就把选定账号写回 IDE 自己的存储，顺带把配额、实例、签到一起管了 🎛️

---

## 2. ✨ 能做什么

| 能力                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| 🔀 **一键切换账号**    | 把账号凭据写回 IDE 的真实存储（`Local State` / `state.vscdb` / `storage.json`），省掉退出再登录；写入前自动备份，失败自动还原 |
| 📊 **配额监控**        | 定时拉取各平台用量，卡片与图表展示剩余额度、重置时间、订阅状态；支持批量刷新、失效识别、自动接管本机最新票据 |
| 🧬 **多账号 / 多实例** | 同一台机器并行跑同一个 IDE 的多个实例，每个实例绑不同账号、独立 user data 目录 |
| 🎁 **每日签到**        | CodeBuddy CN、WorkBuddy CN、Trae CN、Qoder / Qoder CN 支持逐账号签到与一键全签 |
| 🗂️ **账号整理**        | 分组、标签、筛选、JSON 批量导入导出、失效检测、按阈值自动切号 |
| 🧰 **辅助工具**        | 2FA 验证码库、WebDAV 同步、桌面悬浮卡片、日志查看器、Codex 本地 API 网关 |

---

## 3. 🧩 支持的平台

| 分组                    | 平台                                                         |
| ----------------------- | ------------------------------------------------------------ |
| 🛰️ **Antigravity**       | Antigravity、Antigravity IDE                                 |
| 🤖 **Codex**             | Codex、Codex 本地 API 服务                                   |
| 🧠 **Claude**            | Claude（账号管理 + 桌面网关）                                |
| 🐙 **GitHub / 国内大厂** | GitHub Copilot、CodeBuddy CN、WorkBuddy CN、**CatPaw（美团系）** |
| 🦉 **Qoder**             | Qoder（国际版）、**Qoder CN（国内版）**                      |
| 🚄 **Trae**              | Trae、Trae CN（TRAE SOLO 已并入客户端，独立入口移除）        |
| 🧭 **其他**              | Cursor、Windsurf、Kiro、Grok CLI、Zed、ZCode                 |

加粗的是本分支新增的平台 🆕 CodeBuddy 国际版入口已下线，保留的国内版与 WorkBuddy 共用同一套票据体系。

---

## 4. 🔧 这个分支相对上游做了什么

上游 v1.3.35 之外，这个分支主要做了四类事：补两个平台、修通三处「账号莫名掉登录」、把签到真正跑通、把文档从五份合并成三份 🧱

| 改动                                    | 一句话                                                       |
| --------------------------------------- | ------------------------------------------------------------ |
| 🦉 **Qoder CN 支持**                     | 国内版网关是 `openapi.qoder.com.cn` 而不是国际版 `.sh`；登录改走 `client_id` 分支，账号库与实例目录按版本隔离 |
| 🎁 **Qoder 每日签到**                    | 走服务端 campaign 接口领取，每天 10:00（UTC+8）刷新 100 Credits、30 天有效；关键点是对这类请求必须带 `Cosy-ClientType: 10` |
| ♻️ **Qoder 令牌续期**                    | 补上 `deviceToken/refresh`，到期前 60 秒主动续，401 再兜底一次，不再「AT 一过期就永久失效」 |
| 🐾 **CatPaw 平台**                       | 全新接入（账号、配额、实例、切号）；凭据是明文 JSON，不需要解 Safe Storage |
| 📅 **Trae 签到重写**                     | 原实现 `GET + Bearer` 必被风控拒（`9074`），改为 `POST + Cloud-IDE-JWT`；设备号按账号派生，解决「签到按设备限次、只能签第一个号」 |
| 💾 **Trae 刷新后回写**                   | 后台刷新只更新 cockpit 账号库、不回写 `storage.json`，而 refresh token 是轮换的 → 直接双击 IDE 图标会显示未登录。现在刷新成功后按三个前提条件安全回写 |
| 🔓 **WorkBuddy 切号免退出**              | 检测到加密节点改为「备份 → 重建 → 失败还原」，不再要求先手动退出客户端 |
| 🤝 **WorkBuddy / CodeBuddy CN 共用票据** | 两边同一 Keycloak realm，换票后自动回写给同 uid 的兄弟账号，修掉「同步一次就互顶成 12153」 |
| 🩹 **WorkBuddy 刷新自愈**                | 每账号一把异步锁、AT 未过期直接查额度、失败时从本机官方客户端收回最新票据重试，刷新永不向界面抛错 |
| 🧹 **TRAE / SOLO 合并收敛**              | 官方已合并为单客户端，首次加载按 uid 去重、旧账号移入备份目录、入口统一 |
| ⏱️ **Antigravity 去掉 5h 窗口**          | 官方已取消 5 小时限额，卡片从四条降为两条 Weekly，旧数据仍可兜底 |
| 📚 **文档整合**                          | 五份合并为 `CHANGELOG` / `ARCHITECTURE` / `DEPLOYMENT`，并解除编译期对文档的 `include_str!` 依赖 |

逐条根因、试错过程和结论都写在 [docs/CHANGELOG.md](docs/CHANGELOG.md)，不在此重复 👀

---

## 5. 🏗️ 技术架构

三层进程，边界清楚：

```
cockpit-tools.exe（Rust / Tauri 2）
├── React 前端（打包进 exe）──Tauri IPC── Rust 命令层 commands/ modules/
├── 文件系统 / DPAPI ── 读写 IDE 自己的登录态
└── cockpit-cliproxy.exe（Go sidecar，本地 HTTP 网关，可选）
        └── Codex CLI / Claude Code 等外部客户端
```

| 层         | 技术                                                         |
| ---------- | ------------------------------------------------------------ |
| 🎨 界面     | React 19 + TypeScript + Tailwind / daisyUI + zustand + i18next（20+ 语言包） |
| 📦 外壳     | Tauri 2（Rust stable），单 exe 内嵌前端资源                  |
| 🧠 核心逻辑 | `crates/cockpit-core` 供 GUI 与 CLI 共用；`crates/cockpit-cli` 为命令行版 |
| 🌐 本地网关 | Go sidecar（基于 CLIProxyAPI，MIT），Rust 只当看门狗         |
| 💽 存储     | JSON 账号库 + AES-256-GCM 加密凭据；会话与统计走 SQLite      |

**为什么网关是 Go 而不是 Rust**：它要被外部 CLI 直连、要能单独重启、协议转换与流式转发是现成的 SDK。判断标准很简单——需要「被外部程序连上」的能力放 sidecar，只在应用内用的放 Rust 模块 🧭 细节见 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)。

---

## 6. 🚀 编译与运行

**环境**：Windows 10/11 + WebView2；Rust stable（1.88+）；Node 20+；Visual Studio「使用 C++ 的桌面开发」。Go 1.21+ 仅在需要重编 sidecar 时用到（仓库内已带预编译产物）🧰

```bash
npm ci
npm run tauri:dev          # 开发模式
npx tauri build --no-bundle # 出成品
```

产物：

```
target/release/cockpit-tools.exe     主程序
target/release/cockpit-cliproxy.exe  sidecar，必须与主程序放在一起
```

加 `--no-bundle` 是为了跳过打包安装包那一步（会联网拉 WiX/NSIS，国内网络常失败）😅 报 `failed to bundle project` 时不用管，exe 早已生成。

只验证代码是否通过：`npm run typecheck`（秒级）+ `cargo check` ⚡

编译、部署、排障的完整说明在 [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md)。

**关于自动更新**：上游的自动更新会整体覆盖本分支的改动，所以这个分支关闭更新检查、不发布版本 🙅 想跟进上游 = 拉取新版本 → 重新应用改动 → 自己编译。

---

## 7. 🔐 数据与隐私

| 数据             | 位置                                                         |
| ---------------- | ------------------------------------------------------------ |
| 🗂️ 账号索引与配置 | `%USERPROFILE%\.antigravity_cockpit\`                        |
| 🔑 各平台凭据     | 同目录 `<platform>_accounts\`，AES-256-GCM 加密，密钥本机生成、明文不落盘 |
| 📝 日志           | 同目录 `logs\app.log.YYYY-MM-DD`（排障第一步看这里）         |
| 🧳 便携模式       | `<exe 同级>\data\`（放一个 `portable.dat` 即启用）           |

- ✅ 凭据加密；读取 IDE 登录态走 Chromium OSCrypt / Windows DPAPI，绑定「这台机器 + 这个 Windows 用户」，拷到别的电脑解不开。
- 🚫 无云端、无注册、无遥测；只在登录、刷新 token、查询配额时联网。
- ⚠️ Codex 本地网关默认只监听 `127.0.0.1`，改成 `0.0.0.0` 等于对整个局域网开放。
- 📦 备份或分享时不要把整个用户目录打包——里面是 token。

---

## 8. ⚠️ 已知限制

- 🪟 只在 Windows 10/11 上实测过；代码里有 macOS / Linux 分支，但未验证。
- ✍️ 可执行文件未做代码签名，首次运行会被 SmartScreen 拦一次。
- 🔌 部分平台接口来自逆向，官方一次客户端更新就可能失效；失效时的表现通常是「登录已失效」或配额空白，处理办法见 CHANGELOG。
- ⚖️ 单实例、切号、签到等操作都依赖各平台自身的服务条款，账号风险由使用者自行承担。

---

## 9. 📚 文档

| 文档                                         | 回答什么                                                   |
| -------------------------------------------- | ---------------------------------------------------------- |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | 进程边界、模块职责、账号与凭据模型、关键设计决策、已知限制 |
| [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md)     | 环境准备、编译、部署、目录结构、常见问题速查               |
| [docs/CHANGELOG.md](docs/CHANGELOG.md)       | 相对上游改了什么、为什么这么改、踩过哪些坑                 |

读代码建议不要按目录通读，挑一条完整链路走通即可，例如 👇

```
TraeAccountsPage.tsx 点击「刷新」
  → services/traeService.ts                   invoke 封装
  → commands/trae.rs                          Tauri 命令边界
  → modules/trae_account_token_injection.rs   请求与解析
  → 写回 ~/.antigravity_cockpit/trae_accounts.json
  → 前端 store 更新 → 卡片重绘
```

其余平台同构，只是换了名字 ✨

---

## 10. 💛 致谢与许可

### 🙏 致谢

- 上游项目：[jlcodes99/cockpit-tools](https://github.com/jlcodes99/cockpit-tools)
- Antigravity 切号逻辑参考：[Antigravity-Manager](https://github.com/lbjlaq/Antigravity-Manager)
- Codex API 服务集成：[CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI)（MIT）
- Grok 图标造型参考：[LobeHub/lobe-icons](https://github.com/lobehub/lobe-icons)（MIT）

### 📜 许可证

上游采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh-hans)（署名 — 非商业性使用 — 相同方式共享），本分支沿用。

- ✅ **允许**：个人学习、研究等非商业场景下的使用与修改，需保留署名并以相同协议分享
- ❌ **不允许**：任何未获授权的商业使用（含企业内部商业用途、对外商业服务、付费产品集成、二次分发售卖）
- ✉️ **商业授权**：请联系原作者获取书面授权

### ⚖️ 免责声明

本项目仅供个人学习和研究。未获书面商业授权前不得用于商业用途；使用风险自负；请遵守各平台服务条款与当地法律法规。

特别提示：切号、批量签到等操作可能与平台服务条款存在冲突，**账号风险由使用者自行承担** 🙇 本项目不绕过任何付费墙，只做「同一账号在本机更方便地切换」这一件事。
