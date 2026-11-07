像 singbox 一样的丝滑的 Clash 覆写脚本

---

Origin（国家/地区额外分组，Cloudflare 验证规则走 main 代理组）
```text
https://raw.githubusercontent.com/bobvhfhh/sing-mix-custom/refs/heads/main/sing-mix_origin
```

基于 [Sakyvo/sing-mix](https://github.com/Sakyvo/sing-mix) 的 Origin 脚本，仅将 Cloudflare 验证规则的策略从 `DIRECT` 改为 `main`，其余分流逻辑保持原样。

## 自动 AI 更新

GitHub Actions 每周一检查 [上游 Origin 脚本](https://github.com/Sakyvo/sing-mix/blob/main/sing-mix_origin)。发现上游有变化时，会调用 OpenAI API 比较上游与当前脚本，生成择优更新候选；通过 JavaScript 语法检查，并确认以下 Cloudflare 规则仍保留：精确匹配 `challenges.cloudflare.com`、后缀匹配 `cloudflarechallenge.com`，以及验证流量走 `main` 代理组。之后 Action 会把候选脚本和上游快照推到自动分支并创建或更新 PR。

工作流不会自动合并。审查 PR 后，合并才会更新上方 Bettbox 使用的 `main` 脚本。即使 AI 判断某项上游变化不适合，也会在 PR 中说明并提供候选，需人工检查后再决定。

首次使用前，在仓库 **Settings → Secrets and variables → Actions** 中新增 Repository secret：

- `OPENAI_API_KEY`：OpenAI API 密钥。只保存到 GitHub Secret，不要写进文件或提交到仓库。

可选地新增 Repository variable `OPENAI_MODEL` 指定模型；未设置时使用 `gpt-5.4-mini`。OpenAI API 按用量计费，费用取决于模型和上游变更大小。API 密钥与 ChatGPT 订阅分别计费。

可在仓库 **Actions → AI review upstream sing-mix → Run workflow** 手动触发检查。定时任务使用 GitHub Actions；若 Actions 因仓库长期无活动被停用，需要在 Actions 页面重新启用。
