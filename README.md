<div align="center">
  <h1>AirGate Playground</h1>

  <p><strong>平台内置 Web 聊天界面插件（AI Chat）</strong></p>

  <p>
    <a href="https://github.com/DouDOU-start/airgate-playground/releases"><img src="https://img.shields.io/github/v/release/DouDOU-start/airgate-playground?style=flat-square" alt="release" /></a>
    <a href="https://github.com/DouDOU-start/airgate-playground/blob/master/LICENSE"><img src="https://img.shields.io/github/license/DouDOU-start/airgate-playground?style=flat-square" alt="license" /></a>
    <a href="https://github.com/DouDOU-start/airgate-playground/actions/workflows/ci.yml"><img src="https://img.shields.io/github/actions/workflow/status/DouDOU-start/airgate-playground/ci.yml?branch=master&style=flat-square&label=CI" alt="ci" /></a>
    <img src="https://img.shields.io/badge/Go-1.25-00ADD8?style=flat-square&logo=go" alt="go" />
    <img src="https://img.shields.io/badge/React-19-61DAFB?style=flat-square&logo=react" alt="react" />
  </p>
</div>

---

AirGate 扩展插件：内置 Web 聊天界面。安装后用户可在 AirGate 平台内直接与已接入的 AI 模型对话，无需第三方客户端。

- 插件 ID：`airgate-playground` · 类型：`extension`
- 依赖：AirGate Core（经 `Host.Invoke` 调用平台能力，转发走 `gateway.forward`，由 Core 统一调度与计费）

## ✨ 核心特性

- **多模型对话**：自由切换平台已接入的任意模型，流式输出
- **会话管理**：多会话、消息持久化、历史回看
- **多模态输入**：图片上传（经 Core 资产服务存储）
- **思维链展示**：支持推理模型的 reasoning 内容渲染
- **富文本渲染**：Markdown、代码高亮、KaTeX 数学公式
- **余额显示**：实时展示当前用户余额
- **多语言**：界面 i18n（中/英）

## 🧩 接入位置

扩展（extension）插件，自身不直连上游；对话经 Core 转发管线完成：

```text
用户浏览器 → /playground 页面（前端 bundle，Core 提供资产）
                │ 调插件用户 API（/api/v1/ext-user/airgate-playground/*，JWT）
                ▼
        airgate-playground（本仓，gRPC 子进程）
                │ Host.Invoke("gateway.forward") → Core 统一鉴权 / 调度 / 计费
                ▼
        Core 转发管线 → 网关插件 → 上游 AI 平台
```

## 🚦 路由

用户入口 `/api/v1/ext-user/airgate-playground/*`（JWT 鉴权），由 `backend/internal/playground/routes.go` 声明：

| 方法 | 路径 | 说明 |
|---|---|---|
| POST / GET | `/conversations` | 创建 / 列出会话 |
| GET / PUT / DELETE | `/conversations/{id}` | 读取 / 更新 / 删除会话 |
| GET / PUT | `/messages/{id}` | 列出 / 更新消息 |
| POST | `/messages` | 持久化消息 |
| POST | `/chat/completions` | 对话补全（经 `gateway.forward` 流式转发） |
| GET | `/user/info` | 当前用户信息与余额 |

前端单页入口：平台内 `/playground` 页面。

## 🔧 配置

| 配置项 | 说明 |
|---|---|
| `max_conversations_per_user` | 每用户最大会话数限制 |

## 📁 目录结构

```text
backend/   Go 插件实现（internal/playground/：路由、会话/消息存储、Host 调用）
web/       前端（React 19 + Vite），输出 web/dist/index.js
```

## 🚀 构建与开发

```bash
make install   # 安装前后端依赖
make build     # 前端 bundle → 嵌入 → Go 二进制（bin/airgate-playground）
make ci        # lint + test + vet + build
make release   # 交叉编译 linux-amd64
```

产出的二进制由 AirGate Core 作为 gRPC 子进程加载；前端为单 `index.js` bundle，由 Core 统一提供资产服务。开发期建议经 Core 的 `make dev`（含插件 watch）联调。

## 📦 发版

```bash
git tag v0.1.0
git push origin v0.1.0
```

`release.yml` 会自动矩阵构建 4 个平台二进制（linux/darwin × amd64/arm64），通过 ldflags 注入版本号，上传到 GitHub Release。

## 🤝 相关文档

- 开发护栏：[`CLAUDE.md`](CLAUDE.md)
- 插件开发流程：skill `develop-plugin`
- 插件契约与生态架构：skill `core-dev`

## 📜 License

MIT — 详见 [LICENSE](LICENSE)。
