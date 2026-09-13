# Agent Note: Harness 桥接的目录模型

Status: implemented

[English](2026-09-13-harness-bridged-catalog-models.md) | 中文

## Problem

已安装 pi-ai 目录中的 `opencode-go` 路由按发布节奏提供模型 id：端点新增了 `deepseek-v4.1-flash`（DeepSeek V4.1 Flash，1M 上下文 / 384K 最大输出），而已安装的 `@earendil-works/pi-ai` 目录——0.85.1，最新已发布版本——对该路由仍只认识 `deepseek-v4-flash`、`deepseek-v4-pro` 与 `deepseek-v4-flash-vision-exp`。因此 harness 对端点已能服务的模型没有提供任何可选选项。

## Decision

`dsh-llm-pi-ai` 在其自身的目录集成中桥接一小部分目录缺口（`src/catalog.ts` 的 `CATALOG_ADDITIONS`）：按路由与缺失的模型 id 键控，每个条目指名同路由上用于克隆的兄弟条目，因此协议、compat 开关、thinking 映射、容量与定价元数据一并继承，只有 id 与显示名改变。桥接把 `opencode-go/deepseek-v4.1-flash`（从 `deepseek-v4-flash` 克隆）加入 `catalogModels()`，它供模型发现、模型解析与选择器界面使用。已安装目录已经提供的模型留给目录自己的条目，这同时也是上游化信号：当已安装目录带上该 id 时，删除桥接条目、其合并分支及其固定测试。

## Alternatives considered

**pnpm-patch pi-ai 已发布的目录数据。** 修补 `dist/providers/data/opencode-go.json` 可行，但会编辑第三方包中 harness 源码不可见的载荷，折腾锁文件与补丁管道，并与下一次 pi-ai 升级冲突。harness 已经拥有目录集成及其 drift gates，因此一个窄小的 harness 侧桥接与 opencode-go 会话标头修复已添加的路由事实位于同一处。

**仅配置方式（`models` / `modelOverrides`）。** 要求每个部署手写该条目；模型应当与目录兄弟条目一样无需设置即可选择，而且此处不发布任何默认配置。

**升级 pi-ai。** 本仓库已经视「新鲜 pi-ai 发布携带模型目录更新」为常规路径，但 0.85.1 是已发布的最新版本，没有更新的版本携带该 id，因此没有可升级的目标。

## Consequences

OpenCode Go 路由在一切读取目录之处提供 `deepseek-v4.1-flash`，在 pi-ai 提供独立条目之前，其协议行为继承自 `deepseek-v4-flash`。让桥接保持最新是一项小而明确的维护职责：一旦已安装目录携带该 id，目录自己的条目会自动胜出，届时移除桥接条目、其合并分支及其固定测试。

## Related

[提供方路由 LLM 适配器记录](2026-07-14-provider-routed-llm-adapters.zh.md) 拥有此桥接所扩展的路由/目录设计。