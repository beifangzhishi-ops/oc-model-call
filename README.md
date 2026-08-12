# oc-model-call

OpenCode（oc）模型调用指导 skill：统一指导调用 OpenCode 的 GO 订阅与 Zen 按量模型，以及直连 Google Gemini（官方 API），覆盖文本与多模态（图片/音频）场景。

## 功能

- 文本调用：deepseek-v4-flash / pro 等
- 直连 DeepSeek：官方 Responses API（deepseek-v4-flash / deepseek-v4-pro），独立于 oc 中转
- 多模态调用：mimo-v2.5、minimax-m3、qwen3.7-plus、Claude、Gemini、GPT-5.x 等；音频支持直连 Gemini（mp3）与 Zen Gemini 部分模型
- 直连 Gemini：Google 官方 OpenAI 兼容端点（免费层），与 oc 中转的 Zen Gemini 区分
- 费用与授权：顶模调用前先向用户确认模型名、套件、用途与费用
- 故障处理：记录实测的端点、错误码与解决方法

## 安装

在 Codex 中使用 `$skill-installer` 从本仓库安装：

```text
$skill-installer --repo beifangzhishi-ops/oc-model-call --path .
```

或直接克隆到 Codex 本地 skill 目录：

```bash
git clone https://github.com/beifangzhishi-ops/oc-model-call.git ~/.codex/skills/oc-model-call
```

安装后重启 Codex，或等待自动检测生效。

## 使用

- 显式调用：`$oc-model-call`
- 隐式调用：任务匹配 skill 的 description 时自动触发
- 多模态场景：把图片连同要判断的问题一起发送；音频当前仅 Zen Gemini 部分模型与直连 Gemini 支持

## 凭据配置（通用变量）

| 通道 | 变量 | 位置 |
|---|---|---|
| oc（GO/Zen） | `experimental_bearer_token`，或环境变量 `OPENCODE_API_KEY` | `~/.codex/config_opencode.toml` |
| 直连 Gemini（Google 官方 API） | `experimental_bearer_token` | `~/.codex/config_google.toml` |

境外访问如需代理，按本机网络配置添加 `-x` 参数。本仓库不包含、也不会接受任何真实密钥。

## 免责声明

价格表与多模态能力为 2026-08 的个人实测记录，可能过期；请以 OpenCode 官方最新信息为准。

## License

MIT
