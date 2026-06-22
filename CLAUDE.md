# airgate-playground — Claude 开发指南

> 叠加在 monorepo 根 `../CLAUDE.md` 之上。完整流程见共享 skill **`develop-plugin`**；接口契约见 `../airgate-sdk/CLAUDE.md`。

- **插件身份**：id `airgate-playground`，type `extension`，作用 = Web 聊天 UI（"AI Chat"）。
- 实现 `sdk.ExtensionPlugin`：提供 Web 聊天 UI 与自定义 API；经 `Host.Invoke` 的 `gateway.forward` 调用 Core 转发管线完成对话。
- 元信息在 `backend/internal/playground/metadata.go`。

## 🚫 红线

通用边界铁律（只依赖 `airgate-sdk`、经 `Host.Invoke`/`InvokeStream` 调 core、`plugin.yaml` 由 `make manifest` 生成不可手改、前端单 `index.js` bundle）见 skill **`develop-plugin`「🚫 边界铁律」**。

## 混合现状（过渡态）

本仓作为 UI 插件，当前混入了协议转发与任务编排职责（目标应为 UI-only，经 Core 编排 API 工作）：

- **协议转发**：经 `Host.Invoke("gateway.forward")` 转发对话请求（`host_api.go`）
- **SSE 解析**：后端解析上游 SSE 流式响应
- **任务协作**：经 `Host.Invoke` 创建/管理任务、存取资产
- **会话持久化**：本地 SQLite 存储对话历史（`db.go`/`service.go`）

> 新增/改动须按 UI 插件职责归位，勿加深协议/任务编排逻辑。详见 skill `core-dev`「技术债」。

## 命令

构建/发布命令见 skill **`develop-plugin`「构建 / 发布」**；本仓实际 make 目标以 `Makefile` 为准。
