---
name: oc-model-call
description: 调用 OpenCode（oc）模型（GO 订阅与 Zen 按量）与直连 Google Gemini（官方 API，区别于 oc 中转的 Zen Gemini）的统一指导，覆盖 deepseek-v4-flash/pro 等文本模型和 mimo-v2.5、minimax-m3、qwen3.7-plus、gemini-3.6-flash 等多模态模型：端点、认证、请求格式、凭据、代理与降级。当任务需要调用 oc 模型、直连 Gemini、主力模型不支持图像或音频、或需要把图片/截图/图表交给多模态模型分析时使用。
---

# OpenCode（oc）模型调用

需要调用 oc 的模型时按本 skill 操作，文本与多模态统一适用。多模态场景：把图片连同要判断的问题一起发送，直接问模型（如“这张图做得如何”“曲线是否正常”“颜色是否合理”）；仅当确实需要图片内容的文字化版本时才要求转成文字描述。

## 费用与授权

- 调用 zen 或 go 的顶模之前，必须先向用户说明：模型名、套件（go/zen）、用途、预估 token 量与费用量级，得到用户明确授权后才能发起调用。未授权不得调用顶模。
- 每次调用顶模前都必须停下来询问用户；一次询问的授权只对应一次调用，授权不可复用、不可顺延到同一任务的后续调用。即使同一任务需要连续多次顶模调用（如分多段审稿），每一次调用前都要单独询问并取得放行；可以在一次询问中说明后续还需要 N 次调用，但每次调用仍须单独确认。
- 顶模判定：输出单价（每百万输出 token 的价格）高于 2 美元的模型判定为顶模。
- 价格表见下文“oc 模型价格表”；价格无法确认、或模型不在价格表内时，一律按顶模流程处理，先向用户确认。
- 同一模型 ID 在 GO 与 Zen 的价格可能不同（如 deepseek-v4-pro 在 GO 输出 0.87 美元为非顶模，在 Zen 输出 3.48 美元为顶模），按实际调用套件判定。
- 按当前价格表，输出单价 > 2 美元的顶模包括：`grok-4.5`、`glm-5.2`、`glm-5.1`、`glm-5`（Zen）、`kimi-k3`、`kimi-k2.7-code`、`kimi-k2.6`、`kimi-k2.5`（Zen，输出 3）、`qwen3.8-max`、`qwen3.7-max`、`qwen3.7-plus`（GO > 256K 上下文档）、`qwen3.6-plus`（输出 3 或 6）、`claude-fable-5`、`claude-opus-5`、`claude-opus-4-8/4-7/4-6/4-5`、`claude-sonnet-5`、`claude-sonnet-4-6/4-5`、`claude-haiku-4-5`、`gemini-3.6-flash`、`gemini-3.5-flash`、`gemini-3.5-flash-lite`、`gemini-3.1-pro`、`gemini-3-flash`、`gpt-5.6-sol`、`gpt-5.6-terra`、`gpt-5.5`、`gpt-5.5-pro`、`gpt-5.4`、`gpt-5.4-pro`、`gpt-5.4-mini`、`gpt-5.3-codex`、`gpt-5.3-codex-spark`、`gpt-5.2`、`gpt-5.2-codex`、`gpt-5.1`、`gpt-5.1-codex`、`gpt-5.1-codex-max`、`gpt-5`、`gpt-5-codex`、`deepseek-v4-pro`（仅 Zen）。
- 非顶模（无需授权）：`deepseek-v4-flash`、`deepseek-v4-flash-free`、`deepseek-v4-pro`（GO 档）、`mimo-v2.5`、`mimo-v2.5-free`、`mimo-v2.5-pro`、`minimax-m3`、`minimax-m2.7`、`minimax-m2.5`、`gpt-5.6-luna`（含 > 272K 档，输出 1.8 仍低于 2）、`hy3`（GO 档，输出 0.58；Zen 档为 free 的 hy3-free）、`qwen3.7-plus`（GO ≤ 256K 档）、`gpt-5.4-nano`、`gpt-5.1-codex-mini`、`gpt-5-nano`、`grok-build-0.1`、`big-pickle`、`laguna-s-2.1-free`、`ling-3.0-tiny-free`、`nemotron-3-ultra-free`、`nemotron-3.5-lightning-free`。

### 调用失败必须回写本 skill

- 遇到调用失败时，将模型、套件（go/zen）、端点、错误码与错误信息、尝试过的变体、最终解决方法记入本 skill 的“故障处理”一节。
- 如果找到了解决方法，同样写入本 skill，避免下次重复排查。
- 暂时无法解决的失败，也记下“未确认/网关不可用”的状态与日期，后续重试确认后更新。

## oc 模型价格表（美元 / 每百万 token）

以下为 2026-08 实测确认的价格表，可能随时间变化；调用前以 OpenCode 官方最新价格与用户最新确认为准。

### GO 订阅

| 模型 | 输入 | 输出 | 缓存读取 | 缓存写入 | 使用额度 |
|---|---:|---:|---:|---:|---:|
| Grok 4.5 | 2.00 | 6.00 | 0.30 | - | 15 |
| GPT 5.6 Luna（≤ 272K） | 0.20 | 1.20 | 0.02 | 0.25 | 15 |
| GPT 5.6 Luna（> 272K） | 0.40 | 1.80 | 0.04 | 0.50 | 15 |
| GLM-5.2 | 1.40 | 4.40 | 0.26 | - | 60 |
| GLM-5.1 | 1.40 | 4.40 | 0.26 | - | 60 |
| Kimi K3 | 3.00 | 15.00 | 0.30 | - | 15 |
| Kimi K2.7 Code | 0.95 | 4.00 | 0.19 | - | 60 |
| Kimi K2.6 | 0.95 | 4.00 | 0.16 | - | 60 |
| MiMo V2.5 | 0.14 | 0.28 | 0.0028 | - | 60 |
| MiMo V2.5 Pro | 0.435 | 0.87 | 0.003625 | - | 15 |
| MiniMax M3 | 0.30 | 1.20 | 0.06 | - | 60 |
| MiniMax M2.7 | 0.30 | 1.20 | 0.06 | 0.375 | 60 |
| MiniMax M2.5 | 0.30 | 1.20 | 0.06 | 0.375 | 60 |
| Qwen3.8 Max | 2.00 | 6.00 | 0.25 | 2.50 | 15 |
| Qwen3.7 Max | 2.50 | 7.50 | 0.50 | 3.125 | 60 |
| Qwen3.7 Plus（≤ 256K） | 0.40 | 1.60 | 0.04 | 0.50 | 60 |
| Qwen3.7 Plus（> 256K） | 1.20 | 4.80 | 0.12 | 1.50 | 60 |
| Qwen3.6 Plus（≤ 256K） | 0.50 | 3.00 | 0.05 | 0.625 | 60 |
| Qwen3.6 Plus（> 256K） | 2.00 | 6.00 | 0.20 | 2.50 | 60 |
| DeepSeek V4 Pro | 0.435 | 0.87 | 0.003625 | - | 15 |
| DeepSeek V4 Flash | 0.14 | 0.28 | 0.0028 | - | 60 |
| Hy3 | 0.14 | 0.58 | 0.035 | - | 60 |

### Zen 按量

| 模型 | 输入 | 输出 | 缓存读取 | 缓存写入 |
|---|---:|---:|---:|---:|
| Big Pickle | Free | Free | Free | - |
| DeepSeek V4 Flash Free | Free | Free | Free | - |
| MiMo-V2.5 Free | Free | Free | Free | - |
| Hy3 Free | Free | Free | Free | - |
| Laguna S 2.1 Free | Free | Free | Free | - |
| Ling-3.0-tiny Free | Free | Free | Free | - |
| Nemotron 3 Ultra Free | Free | Free | Free | - |
| Nemotron 3.5 Lightning Free | Free | Free | Free | - |
| MiniMax M3 | 0.30 | 1.20 | 0.06 | - |
| MiniMax M2.7 | 0.30 | 1.20 | 0.06 | - |
| MiniMax M2.5 | 0.30 | 1.20 | 0.06 | - |
| GLM 5.2 | 1.40 | 4.40 | 0.26 | - |
| GLM 5.1 | 1.40 | 4.40 | 0.26 | - |
| GLM 5 | 1.00 | 3.20 | 0.20 | - |
| Kimi K2.7 Code | 0.95 | 4.00 | 0.19 | - |
| Kimi K3 | 3.00 | 15.00 | 0.30 | - |
| Kimi K2.6 | 0.95 | 4.00 | 0.16 | - |
| Kimi K2.5 | 0.60 | 3.00 | 0.10 | - |
| Qwen3.7 Max | 2.50 | 7.50 | 0.50 | 3.125 |
| Qwen3.7 Plus | 0.40 | 1.60 | 0.04 | 0.50 |
| Qwen3.6 Plus | 0.50 | 3.00 | 0.05 | 0.625 |
| Qwen3.5 Plus | 0.20 | 1.20 | 0.02 | 0.25 |
| DeepSeek V4 Pro | 1.74 | 3.48 | 0.145 | - |
| DeepSeek V4 Flash | 0.14 | 0.28 | 0.028 | - |
| Claude Fable 5 | 10.00 | 50.00 | 1.00 | 12.50 |
| Claude Opus 5 | 5.00 | 25.00 | 0.50 | 6.25 |
| Claude Opus 4.8 | 5.00 | 25.00 | 0.50 | 6.25 |
| Claude Opus 4.7 | 5.00 | 25.00 | 0.50 | 6.25 |
| Claude Opus 4.6 | 5.00 | 25.00 | 0.50 | 6.25 |
| Claude Opus 4.5 | 5.00 | 25.00 | 0.50 | 6.25 |
| Claude Sonnet 5 | 2.00 | 10.00 | 0.20 | 2.50 |
| Claude Sonnet 4.6 | 3.00 | 15.00 | 0.30 | 3.75 |
| Claude Sonnet 4.5（≤ 200K） | 3.00 | 15.00 | 0.30 | 3.75 |
| Claude Sonnet 4.5（> 200K） | 6.00 | 22.50 | 0.60 | 7.50 |
| Claude Haiku 4.5 | 1.00 | 5.00 | 0.10 | 1.25 |
| Gemini 3.6 Flash | 1.50 | 7.50 | 0.15 | - |
| Gemini 3.5 Flash | 1.50 | 9.00 | 0.15 | - |
| Gemini 3.5 Flash Lite | 0.30 | 2.50 | 0.03 | - |
| Gemini 3.1 Pro（≤ 200K） | 2.00 | 12.00 | 0.20 | - |
| Gemini 3.1 Pro（> 200K） | 4.00 | 18.00 | 0.40 | - |
| Gemini 3 Flash | 0.50 | 3.00 | 0.05 | - |
| Grok 4.5（≤ 200K） | 2.00 | 6.00 | 0.30 | - |
| Grok 4.5（> 200K） | 4.00 | 12.00 | 0.60 | - |
| Grok Build 0.1 | 1.00 | 2.00 | 0.20 | - |
| GPT 5.6 Sol（≤ 272K） | 5.00 | 30.00 | 0.50 | 6.25 |
| GPT 5.6 Sol（> 272K） | 10.00 | 45.00 | 1.00 | 12.50 |
| GPT 5.6 Terra（≤ 272K） | 2.00 | 12.00 | 0.20 | 2.50 |
| GPT 5.6 Terra（> 272K） | 4.00 | 18.00 | 0.40 | 5.00 |
| GPT 5.6 Luna（≤ 272K） | 0.20 | 1.20 | 0.02 | 0.25 |
| GPT 5.6 Luna（> 272K） | 0.40 | 1.80 | 0.04 | 0.50 |
| GPT 5.5（≤ 272K） | 5.00 | 30.00 | 0.50 | - |
| GPT 5.5（> 272K） | 10.00 | 45.00 | 1.00 | - |
| GPT 5.5 Pro | 30.00 | 180.00 | 30.00 | - |
| GPT 5.4（≤ 272K） | 2.50 | 15.00 | 0.25 | - |
| GPT 5.4（> 272K） | 5.00 | 22.50 | 0.50 | - |
| GPT 5.4 Pro | 30.00 | 180.00 | 30.00 | - |
| GPT 5.4 Mini | 0.75 | 4.50 | 0.075 | - |
| GPT 5.4 Nano | 0.20 | 1.25 | 0.02 | - |
| GPT 5.3 Codex Spark | 1.75 | 14.00 | 0.175 | - |
| GPT 5.3 Codex | 1.75 | 14.00 | 0.175 | - |
| GPT 5.2 | 1.75 | 14.00 | 0.175 | - |
| GPT 5.2 Codex | 1.75 | 14.00 | 0.175 | - |
| GPT 5.1 | 1.07 | 8.50 | 0.107 | - |
| GPT 5.1 Codex | 1.07 | 8.50 | 0.107 | - |
| GPT 5.1 Codex Max | 1.25 | 10.00 | 0.125 | - |
| GPT 5.1 Codex Mini | 0.25 | 2.00 | 0.025 | - |
| GPT 5 | 1.07 | 8.50 | 0.107 | - |
| GPT 5 Codex | 1.07 | 8.50 | 0.107 | - |
| GPT 5 Nano | 0.05 | 0.40 | 0.005 | - |

## 接口与路由

### 通用约定

- 凭据：oc 从 `~/.codex/config_opencode.toml` 提取 `experimental_bearer_token`（也可用环境变量 `OPENCODE_API_KEY`）；直连 Gemini 从 `~/.codex/config_google.toml` 提取 `experimental_bearer_token`。禁止硬编码、禁止打印。
- 工具：使用 curl.exe（PowerShell 的 curl 别名不可用）；Python 脚本请参考“故障处理”中的 UA 设置。
- 基础端点：GO 订阅 `https://opencode.ai/zen/go/v1`，Zen 按量 `https://opencode.ai/zen/v1`。
- 请求体写入临时 JSON 文件再用 `--data-binary "@file"` 发送，避免命令行转义问题。

### GO 接口（2026-08 实测）

| 模型 | 模型 ID | 端点 | AI SDK 包 |
|---|---|---|---|
| Grok 4.5 | grok-4.5 | https://opencode.ai/zen/go/v1/chat/completions | @ai-sdk/openai-compatible |
| GPT 5.6 Luna | gpt-5.6-luna | https://opencode.ai/zen/go/v1/responses | @ai-sdk/openai |
| GLM-5.2 | glm-5.2 | https://opencode.ai/zen/go/v1/chat/completions | @ai-sdk/openai-compatible |
| GLM-5.1 | glm-5.1 | https://opencode.ai/zen/go/v1/chat/completions | @ai-sdk/openai-compatible |
| Kimi K3 | kimi-k3 | https://opencode.ai/zen/go/v1/chat/completions | @ai-sdk/openai-compatible |
| Kimi K2.7 Code | kimi-k2.7-code | https://opencode.ai/zen/go/v1/chat/completions | @ai-sdk/openai-compatible |
| Kimi K2.6 | kimi-k2.6 | https://opencode.ai/zen/go/v1/chat/completions | @ai-sdk/openai-compatible |
| DeepSeek V4 Pro | deepseek-v4-pro | https://opencode.ai/zen/go/v1/chat/completions | @ai-sdk/openai-compatible |
| DeepSeek V4 Flash | deepseek-v4-flash | https://opencode.ai/zen/go/v1/chat/completions | @ai-sdk/openai-compatible |
| MiMo-V2.5 | mimo-v2.5 | https://opencode.ai/zen/go/v1/chat/completions | @ai-sdk/openai-compatible |
| MiMo-V2.5-Pro | mimo-v2.5-pro | https://opencode.ai/zen/go/v1/chat/completions | @ai-sdk/openai-compatible |
| MiniMax M3 | minimax-m3 | https://opencode.ai/zen/go/v1/messages | @ai-sdk/anthropic |
| MiniMax M2.7 | minimax-m2.7 | https://opencode.ai/zen/go/v1/messages | @ai-sdk/anthropic |
| MiniMax M2.5 | minimax-m2.5 | https://opencode.ai/zen/go/v1/messages | @ai-sdk/anthropic |
| Qwen3.8 Max | qwen3.8-max | https://opencode.ai/zen/go/v1/messages | @ai-sdk/anthropic |
| Qwen3.7 Max | qwen3.7-max | https://opencode.ai/zen/go/v1/messages | @ai-sdk/anthropic |
| Qwen3.7 Plus | qwen3.7-plus | https://opencode.ai/zen/go/v1/messages | @ai-sdk/anthropic |
| Qwen3.6 Plus | qwen3.6-plus | https://opencode.ai/zen/go/v1/messages | @ai-sdk/anthropic |
| Hy3 | hy3 | https://opencode.ai/zen/go/v1/chat/completions | @ai-sdk/openai-compatible |

### Zen 接口（2026-08 实测）

| 模型 | 模型 ID | 端点 | AI SDK 包 |
|---|---|---|---|
| GPT 5.6 Sol | gpt-5.6-sol | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.6 Terra | gpt-5.6-terra | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.6 Luna | gpt-5.6-luna | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.5 | gpt-5.5 | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.5 Pro | gpt-5.5-pro | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.4 | gpt-5.4 | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.4 Pro | gpt-5.4-pro | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.4 Mini | gpt-5.4-mini | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.4 Nano | gpt-5.4-nano | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.3 Codex | gpt-5.3-codex | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.3 Codex Spark | gpt-5.3-codex-spark | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.2 | gpt-5.2 | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.2 Codex | gpt-5.2-codex | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.1 | gpt-5.1 | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.1 Codex | gpt-5.1-codex | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.1 Codex Max | gpt-5.1-codex-max | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5.1 Codex Mini | gpt-5.1-codex-mini | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5 | gpt-5 | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5 Codex | gpt-5-codex | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| GPT 5 Nano | gpt-5-nano | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| Claude Fable 5 | claude-fable-5 | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| Claude Opus 5 | claude-opus-5 | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| Claude Opus 4.8 | claude-opus-4-8 | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| Claude Opus 4.7 | claude-opus-4-7 | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| Claude Opus 4.6 | claude-opus-4-6 | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| Claude Opus 4.5 | claude-opus-4-5 | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| Claude Sonnet 5 | claude-sonnet-5 | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| Claude Sonnet 4.6 | claude-sonnet-4-6 | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| Claude Sonnet 4.5 | claude-sonnet-4-5 | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| Claude Haiku 4.5 | claude-haiku-4-5 | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| Gemini 3.6 Flash | gemini-3.6-flash | https://opencode.ai/zen/v1/models/gemini-3.6-flash | @ai-sdk/google |
| Gemini 3.5 Flash | gemini-3.5-flash | https://opencode.ai/zen/v1/models/gemini-3.5-flash | @ai-sdk/google |
| Gemini 3.5 Flash Lite | gemini-3.5-flash-lite | https://opencode.ai/zen/v1/models/gemini-3.5-flash-lite | @ai-sdk/google |
| Gemini 3.1 Pro | gemini-3.1-pro | https://opencode.ai/zen/v1/models/gemini-3.1-pro | @ai-sdk/google |
| Gemini 3 Flash | gemini-3-flash | https://opencode.ai/zen/v1/models/gemini-3-flash | @ai-sdk/google |
| Grok 4.5 | grok-4.5 | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| Grok Build 0.1 | grok-build-0.1 | https://opencode.ai/zen/v1/responses | @ai-sdk/openai |
| Qwen3.7 Max | qwen3.7-max | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| Qwen3.7 Plus | qwen3.7-plus | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| Qwen3.6 Plus | qwen3.6-plus | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| Qwen3.5 Plus | qwen3.5-plus | https://opencode.ai/zen/v1/messages | @ai-sdk/anthropic |
| DeepSeek V4 Pro | deepseek-v4-pro | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| DeepSeek V4 Flash | deepseek-v4-flash | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| MiniMax M3 | minimax-m3 | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| MiniMax M2.7 | minimax-m2.7 | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| MiniMax M2.5 | minimax-m2.5 | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| GLM 5.2 | glm-5.2 | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| GLM 5.1 | glm-5.1 | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| GLM 5 | glm-5 | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| Kimi K2.5 | kimi-k2.5 | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| Kimi K2.6 | kimi-k2.6 | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| Kimi K2.7 Code | kimi-k2.7-code | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| Kimi K3 | kimi-k3 | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| Big Pickle | big-pickle | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| MiMo-V2.5 Free | mimo-v2.5-free | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| Hy3 Free | hy3-free | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| Laguna S 2.1 Free | laguna-s-2.1-free | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| Ling-3.0-tiny Free | ling-3.0-tiny-free | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| Nemotron 3 Ultra Free | nemotron-3-ultra-free | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| Nemotron 3.5 Lightning Free | nemotron-3.5-lightning-free | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |
| DeepSeek V4 Flash Free | deepseek-v4-flash-free | https://opencode.ai/zen/v1/chat/completions | @ai-sdk/openai-compatible |

## 直连 DeepSeek（官方 API，区别于 oc 中转的 GO/Zen DeepSeek）

- 定位：DeepSeek 官方直连，与 oc 中转互相独立；凭据从 `~/.codex/config_deepseek.toml` 提取 `experimental_bearer_token`，禁止硬编码、禁止打印。
- 端点：`https://api.deepseek.com/responses` 与 `https://api.deepseek.com/v1/responses` 均原生支持 OpenAI Responses API；官方文档已明确列出 `deepseek-v4-pro` 支持（2026-08-13 核对「模型 & 价格」与「使用 Responses API」指南，见 https://api-docs.deepseek.com/zh-cn/guides/responses_api/，model 可选 deepseek-v4-flash / deepseek-v4-pro，`previous_response_id` 不支持），且明确说明该 API“为了满足大家对 Codex 的需求”新增、配置后可直接在 Codex 中使用 DeepSeek 模型（接入 Codex 页的 models.json 同时声明 flash 与 pro），custom 工具仅支持 `apply_patch`（用于 Codex 兼容）；同日本机实测 `deepseek-v4-pro` 成功（HTTP 200，返回 reasoning + message）。注意：官方「更新日志」页仍是 Flash 发布时的旧条目（写 Pro 未改、将尽快发布），核对支持情况以「模型 & 价格」「Responses API 指南」「接入 Codex」为准。
- 多轮 agent 循环（2026-08-13 实测 deepseek-v4-pro）：首轮返回 reasoning + function_call；回传时必须把 `function_call` 作为 input 顶层项（不是嵌在 assistant content 里），紧随 `function_call_output`，第二轮返回最终 message。官方文档明确：input 顶层 `function_call` 会归并到相邻 assistant 消息；`previous_response_id` 不支持（stateless）。
- 请求格式：Responses API（model + input + max_output_tokens），示例：

```json
{
  "model": "deepseek-v4-pro",
  "input": [
    {
      "role": "user",
      "content": [
        {"type": "input_text", "text": "你的问题"}
      ]
    }
  ],
  "max_output_tokens": 2048
}
```

发送：

```powershell
curl.exe -sS --max-time 120 -X POST https://api.deepseek.com/v1/responses -H "Authorization: Bearer <TOKEN>" -H "Content-Type: application/json" --data-binary "@request.json"
```

- 注意：请求 JSON 文件必须为无 BOM 的 UTF-8，否则 curl 发送后返回 400 `Failed to parse the request body as JSON`。
- GO `deepseek-v4-pro` 的 `/responses` 是 chat 映射而非原生透传（2026-08-13 实测）：返回精简 response 壳（无 reasoning、无 status/completed_at 等原生字段，仅 output + stop_reason + usage）；标准 Responses 多轮格式（顶层 function_call）回传报上游 chat 错误 `messages[1]: missing field id`，改用 chat 风格 `assistant.tool_calls` 历史才能继续；错误信息明确暴露 `Error from provider (Console Go)`。GO 的 `/chat/completions` 在 thinking mode 下第二轮必须回传 `reasoning_content`，否则 400。

## 直连 Gemini（Google 官方 API，区别于 oc 中转的 Zen Gemini）

- 定位：Gemini 只负责音频多模态与对话生成，不参与 Codex agent 主力（主力仍为 oc deepseek-v4-flash）。
- 凭据：从 `~/.codex/config_google.toml` 提取 `experimental_bearer_token`，禁止硬编码、禁止打印。
- 端点：`https://generativelanguage.googleapis.com/v1beta/openai/`（OpenAI 兼容），只支持 chat/completions；原生 `/v1beta/models/{id}:generateContent` 端点对该 key 返回 401 `API_KEY_SERVICE_BLOCKED`，不要使用。
- 认证：`Authorization: Bearer <KEY>` 即可。
- 模型：`gemini-3.6-flash`（免费层，输入输出免费，有每日/每分钟限额）。
- 网络：大陆访问需走本机代理，代理地址按本机配置（示例：`-x <PROXY>`）。
- 费用与授权：免费层输出单价 0 美元，不属于顶模，无需授权；额度有限，调用前仍说明用途。

### 文本调用（2026-08-12 实测）

```json
{
  "model": "gemini-3.6-flash",
  "messages": [{"role": "user", "content": "你的问题"}],
  "max_tokens": 2048
}
```

```powershell
curl.exe -sS --max-time 120 -x <PROXY> -X POST https://generativelanguage.googleapis.com/v1beta/openai/chat/completions -H "Authorization: Bearer <KEY>" -H "Content-Type: application/json" --data-binary "@request.json"
```

### 音频调用（2026-08-12 实测：3MB mp3 正确识别歌曲风格/语言/主题；567KB 34s mp3 正确识别歌名；WAV 不识别）

```json
{
  "model": "gemini-3.6-flash",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "这段音频唱的是什么？请描述歌曲风格、语言和内容主题。"},
        {"type": "input_audio", "input_audio": {"data": "<BASE64>", "format": "mp3"}}
      ]
    }
  ],
  "max_tokens": 600
}
```

### 图片调用（OpenAI 兼容格式，与 oc OpenAI 兼容格式一致；直连图片尚未实测）

```json
{
  "model": "gemini-3.6-flash",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "这张图曲线是否正常？"},
        {"type": "image_url", "image_url": {"url": "data:image/png;base64,<BASE64>"}}
      ]
    }
  ],
  "max_tokens": 1024
}
```

## 多模态能力（2026-08-12 最小实测）

测试方法：64×64 PNG（红色圆形带数字 1）+ 0.6 秒 440 Hz WAV，每模型每模态一个最小请求。HTTP 200 且模型能描述图中内容/音频音色 = 支持；400 报错或模型声称看不到附件 = 不支持；503/401 等 = 网关暂不可用或未提供该模型。

### 图片支持

GO：

| 模型 | 图片 | 备注 |
|---|---|---|
| gpt-5.6-luna | 支持 | responses，正确描述 |
| deepseek-v4-flash / deepseek-v4-pro | 不支持 | 纯文本 |
| grok-4.5 | 未确认 | 多次 503 Endpoint unavailable |
| glm-5.2 / glm-5.1 | 不支持 | 200 但模型明示收不到图片 |
| kimi-k3 | 支持 | 正确描述（可审图） |
| kimi-k2.7-code / kimi-k2.6 | 支持 | |
| mimo-v2.5 | 支持 | |
| mimo-v2.5-pro | 不支持 | 400 无图片端点 |
| minimax-m3 | 支持 | |
| qwen3.8-max | 支持 | |
| qwen3.7-max | 不可用 | 400 无错误详情（messages 与 chat 均异常） |
| qwen3.7-plus / qwen3.6-plus | 支持 | |
| hy3 | 不支持 | 400 无图片端点 |

Zen：

| 模型 | 图片 | 备注 |
|---|---|---|
| minimax-m3 | 支持 | |
| glm-5 / glm-5.1 / glm-5.2 | 不支持 | 400 This model does not support image inputs |
| kimi-k3 / kimi-k2.7-code / kimi-k2.6 / kimi-k2.5 | 支持 | |
| big-pickle | 不支持 | 纯文本，image_url 反序列化失败 |
| mimo-v2.5-free | 支持 | |
| hy3-free | 不支持 | 200 但附件被剥离，模型称未收到图片 |
| laguna-s-2.1-free / ling-3.0-tiny-free / nemotron-3-ultra-free / nemotron-3.5-lightning-free | 不支持 | 400 无图片端点 |
| claude-fable-5 / claude-opus-5 / claude-opus-4-8 / 4-7 / 4-6 / 4-5 / claude-sonnet-5 / 4-6 / 4-5 / claude-haiku-4-5 | 支持 | 全部正确描述 |
| qwen3.7-max / qwen3.7-plus | 不可用 | 401 ModelError not supported（网关未提供） |
| qwen3.6-plus / qwen3.5-plus | 支持 | |
| gpt-5.6-sol / terra / luna、gpt-5.5 / 5.5-pro、gpt-5.4 / 5.4-pro / 5.4-mini / 5.4-nano、gpt-5.3-codex、gpt-5.2 / 5.2-codex、gpt-5.1 / 5.1-codex / 5.1-codex-max / 5.1-codex-mini、gpt-5 / 5-codex / 5-nano | 支持 | gpt-5-nano、gpt-5.1-codex-mini 无可见文本（reasoning 加密），其余正确描述 |
| gpt-5.3-codex-spark | 不支持 | 400 does not support image inputs |
| grok-4.5 | 不支持 | responses 400 invalid request；纯文本可用 |
| grok-build-0.1 | 支持 | 正确描述 |
| gemini-3-flash / 3.1-pro / 3.5-flash / 3.5-flash-lite / 3.6-flash | 支持 | 需 x-goog-api-key 认证 |

### 声音支持

| 模型 | 声音 | 备注 |
|---|---|---|
| gemini-3.5-flash / gemini-3.5-flash-lite / gemini-3.6-flash（Zen） | 支持 | 识别为纯音/提示音，频率判断有偏差 |
| gemini-3.6-flash（直连） | 支持 | 3MB mp3 正确识别歌曲风格/语言/主题（2026-08-12 实测） |
| gemini-3-flash / gemini-3.1-pro（Zen） | 不支持 | 200 但只收到音频占位符文本 |
| 其余全部已测模型（GO 与 Zen） | 不支持 | 见下方备注 |

音频不支持明细：go kimi-k3（400 invalid part type input_audio）、go glm-5.2（200 但模型无音频能力）、go mimo-v2.5（200 但音频被网关转成乱码/占位符，模型称收到“[audio]”或“一串问号”；440 Hz 与 1000 Hz 对照均无法区分，reasoning 中“看到波形图”为幻觉）、go hy3 / go mimo-v2.5-pro / zen laguna / zen ling（400 无音频端点）、zen kimi-k3 / zen glm-5.2（400 要求 content 为字符串）、zen big-pickle（400 unknown variant input_audio）、zen hy3-free / zen mimo-v2.5-free（200 但模型称未收到音频）、zen grok-4.5（400 invalid input_audio）、zen grok-build-0.1（422 upstream failed）、go grok-4.5（503 未确认）。

## 文本调用示例（deepseek-v4-flash）

请求体：

```json
{
  "model": "deepseek-v4-flash",
  "messages": [{"role": "user", "content": "你的问题"}],
  "max_tokens": 2048
}
```

发送：

```powershell
curl.exe -sS --max-time 120 -X POST https://opencode.ai/zen/go/v1/chat/completions -H "Authorization: Bearer <TOKEN>" -H "Content-Type: application/json" --data-binary "@request.json"
```

## 图片调用示例

图片先压缩（长边超过 2000px 再转 base64）。

### OpenAI 兼容格式（mimo-v2.5、kimi-k3、grok、GLM 等 chat/completions）

```json
{
  "model": "mimo-v2.5",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "这张图曲线是否正常？"},
        {"type": "image_url", "image_url": {"url": "data:image/png;base64,<BASE64>"}}
      ]
    }
  ],
  "max_tokens": 1024
}
```

### Anthropic 兼容格式（GO minimax-m3、qwen3.7-plus 等 /messages）

```json
{
  "model": "minimax-m3",
  "max_tokens": 1024,
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "这张图配色如何？"},
        {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": "<BASE64>"}}
      ]
    }
  ]
}
```

发送时同时带两个认证头：

```powershell
curl.exe -sS --max-time 120 -X POST https://opencode.ai/zen/go/v1/messages -H "x-api-key: <TOKEN>" -H "Authorization: Bearer <TOKEN>" -H "Content-Type: application/json" --data-binary "@request.json"
```

### Responses 格式（GPT 5.x、Zen grok 等 /responses）

```json
{
  "model": "gpt-5.6-luna",
  "input": [
    {
      "role": "user",
      "content": [
        {"type": "input_text", "text": "这张图曲线是否正常？"},
        {"type": "input_image", "image_url": "data:image/png;base64,<BASE64>"}
      ]
    }
  ],
  "max_output_tokens": 1024
}
```

### Gemini 格式（Zen /models/{id}:generateContent）

```json
{
  "contents": [
    {
      "role": "user",
      "parts": [
        {"text": "这张图曲线是否正常？"},
        {"inline_data": {"mime_type": "image/png", "data": "<BASE64>"}}
      ]
    }
  ]
}
```

认证必须加 `x-goog-api-key` 头：

```powershell
curl.exe -sS --max-time 120 -X POST https://opencode.ai/zen/v1/models/gemini-3.5-flash:generateContent -H "Authorization: Bearer <TOKEN>" -H "x-goog-api-key: <TOKEN>" -H "Content-Type: application/json" --data-binary "@request.json"
```

### 音频格式（OpenAI 兼容；实测当前仅 Gemini 部分模型支持）

```json
{
  "model": "mimo-v2.5",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "这段音频是什么声音？"},
        {"type": "input_audio", "input_audio": {"data": "<BASE64>", "format": "wav"}}
      ]
    }
  ],
  "max_tokens": 512
}
```

Gemini 音频用 `{"inline_data": {"mime_type": "audio/wav", "data": "<BASE64>"}}`。

## 故障处理（实测记录，2026-08-12）

- 大批量图片请求会触发网关 500：2026-08-12 实测中，GO mimo-v2.5 一次请求携带 5 张大图返回 `{"type":"error","error":{"type":"error","message":"Internal server error"}}`；对照测试证明 Zen mimo-v2.5-free 同批图同样 500，而 GO mimo-v2.5 单张小图成功。根因是一次请求携带过多/过大的图片触发网关 500，不是模型不可用，也不是端点临时故障。
- 解决：多图分析改为逐张调用，每张图独立请求并分别保存响应，再汇总结果；必要时同时减小图片尺寸。
- Cloudflare 403 error 1010：Python urllib 默认 User-Agent 会被拦截。解决：使用 curl.exe（直连即可），或 Python 请求设置 `User-Agent: curl/8.4.0`（实测有效）。不要先怀疑代理。
- curl.exe 从 Python subprocess 启动报 `SEC_E_NO_CREDENTIALS (0x8009030e)`（schannel 凭据错误）：解决：改回 Python urllib 并伪装 curl UA。
- Gemini 401 `Missing API key`：只带 Bearer 不够，必须额外带 `x-goog-api-key: <TOKEN>` 头。
- 直连 Gemini 原生 generateContent 端点 401 `API_KEY_SERVICE_BLOCKED`：该凭据仅支持 OpenAI 兼容端点（`/v1beta/openai/chat/completions` + Bearer），统一走 OpenAI 兼容格式。
- 直连 Gemini 不带代理时连接失败：大陆访问必须走本机代理（代理地址见本机配置，不写入公开文档）。
- 直连 Gemini OpenAI 兼容端点音频：16-bit PCM WAV（440/880 Hz 双音）返回 200 但模型回复“消息未传清楚”，音频未被识别；同一端点 mp3 正常（实测 567KB 34s 歌曲正确识别歌名）。音频任务统一使用 mp3，避免 WAV。
- `qwen3.7-max`（GO）：messages 端点返回 400 且错误体只有 `{"model":"qwen3.7-max"}`，chat/completions 同样 400；`qwen3.7-max` / `qwen3.7-plus`（Zen）：401 `Model qwen3.7-max is not supported`。结论：网关当前未提供可用端点，勿误用；调用前先向用户说明。
- `grok-4.5`（GO）：多次重试均 503 `Endpoint is unavailable`；`grok-4.5`（Zen）：文本可用，图片 400、音频 400。需按此记录，恢复后更新。
- GLM 系列：GO 端 200 但图片/音频被剥离（模型称无多模态能力），Zen 端直接 400 `does not support image inputs`。两套件均按不支持处理。
- 免费模型 hy3-free：200 但附件被剥离成占位符（模型称“未收到图片/音频”），判定不支持。
- 遇到任何新的调用失败或解决方法，按“调用失败必须回写本 skill”一节补充到本节。

## 注意事项

- 图片会发送到 OpenCode 上游模型服务，不要发送未授权或敏感图片。
- 请求消耗 GO 订阅额度或 Zen 按量费用；高分辨率图片 token 消耗更大。
- 顶模调用必须按“费用与授权”一节取得用户授权；授权后仍应在请求前复核模型名与套件，避免误用高价模型。
- 音频实测中仅 Zen Gemini 3.5-flash / 3.5-flash-lite / 3.6-flash 可用；其余模型按文本处理，不要把音频任务派给它们。
- kimi-k3 在 oc 网关可看图（2026-08-12 实测），但不可听音频；审图任务可用 k3，审音频任务请改用 Zen Gemini。
- 直连 Gemini（Google 官方）与 Zen Gemini（oc 中转）是两套独立通道：直连走 OpenAI 兼容端点 + `config_google.toml` 凭据 + 免费层（无需授权）；Zen 走 opencode.ai + Google 格式 + `x-goog-api-key`，按量计费且属顶模需授权。
