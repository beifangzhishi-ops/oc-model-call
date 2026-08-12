# oc-model-call 维护规则

本项目不受全局规则中“.codex 不用 git 维护”这一规则的限制，需要 git 维护并同步推送到云端。

本 skill 为通用技能，文档中禁止写死本机用户名、机器名（如 game）、个人项目名等非通用内容；发现后需泛化为通用表述。

通用变量（凭据与网络约定）统一在 SKILL.md 与 README.md 中说明：

- oc 凭据：`~/.codex/config_opencode.toml` 的 `experimental_bearer_token`，或环境变量 `OPENCODE_API_KEY`；
- 直连 Gemini 凭据：`~/.codex/config_google.toml` 的 `experimental_bearer_token`；
- 本机代理约定：`http://127.0.0.1:7890`。

禁止硬编码、禁止打印、禁止提交任何真实凭据；修改文档后需同步更新 README.md 与 SKILL.md 的对应说明。
