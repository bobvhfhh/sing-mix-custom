像 singbox 一样的丝滑的 Clash 覆写脚本

---

Origin（国家/地区额外分组，Cloudflare 验证规则走 main 代理组）
```text
https://raw.githubusercontent.com/bobvhfhh/sing-mix-custom/refs/heads/main/sing-mix_origin
```

基于 [Sakyvo/sing-mix](https://github.com/Sakyvo/sing-mix) 的 Origin 脚本，仅将 Cloudflare 验证规则的策略从 `DIRECT` 改为 `main`，其余分流逻辑保持原样。

## 自动 AI 更新

GitHub Actions 每周一检查 [上游 Origin 脚本](https://github.com/Sakyvo/sing-mix/blob/main/sing-mix_origin)。发现上游有变化时，会调用你配置的 AI 服务比较上游与当前脚本，生成择优更新候选；通过 JavaScript 语法检查，并确认以下 Cloudflare 规则仍保留：精确匹配 `challenges.cloudflare.com`、后缀匹配 `cloudflarechallenge.com`，以及验证流量走 `main` 代理组。之后 Action 会把候选脚本和上游快照推到自动分支并创建或更新 PR。

工作流不会自动合并。审查 PR 并合并后，只会更新 GitHub 上方 Raw 链接对应的 `main` 脚本；Bettbox 中已保存的配置不会因此自动变化。要让新脚本在 Bettbox 生效，还需要在 Bettbox 中手动点击“同步”，让它通过该链接重新拉取最新内容。即使 AI 判断某项上游变化不适合，也会在 PR 中提供候选，需人工检查后再决定。

该工作流使用 OpenAI Chat Completions 兼容接口，所以可接入支持此接口的中转服务或 DeepSeek。先确认你的服务商提供兼容的 `/chat/completions` 路径、Bearer 密钥认证和 `choices[0].message.content` 响应格式。

首次使用前，在仓库 **Settings → Secrets and variables → Actions** 中配置：

- Repository secret `AI_API_KEY`：所选服务的 API 密钥。只保存到 GitHub Secret，不要写进文件或提交到仓库。
- Repository variable `AI_API_BASE_URL`：服务商给出的 HTTPS API 根地址，不带 `/chat/completions`。例如 DeepSeek 的根地址是 `https://api.deepseek.com`；中转服务请使用其文档提供的地址。
- Repository variable `AI_MODEL`：服务商支持的模型 ID。可填 DeepSeek 文档给出的模型名，或中转服务提供的模型 ID。

使用 DeepSeek 时，按 DeepSeek API 文档配置 Base URL、模型 ID 和对应 API Key；使用中转站时同样填写中转站提供的三项信息。GitHub Actions 会在检测到上游变更后将两份脚本发送给所配置的服务处理，因此应选用你信任的服务，并留意其数据和计费政策。API 用量由该 API 服务商计费，ChatGPT 订阅通常不包含 API 调用额度。

可在仓库 **Actions → AI review upstream sing-mix → Run workflow** 手动触发检查。定时任务使用 GitHub Actions；若 Actions 因仓库长期无活动被停用，需要在 Actions 页面重新启用。
