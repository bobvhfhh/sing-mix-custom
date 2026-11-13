像 singbox 一样的丝滑的 Clash 覆写脚本

---

Origin（国家/地区额外分组，Cloudflare 验证规则走 main 代理组）
```text
https://raw.githubusercontent.com/bobvhfhh/sing-mix-custom/refs/heads/main/sing-mix_origin
```

基于 [Sakyvo/sing-mix](https://github.com/Sakyvo/sing-mix) 的 Origin 脚本，仅将 Cloudflare 验证规则的策略从 `DIRECT` 改为 `main`，其余分流逻辑保持原样。

脚本包含两条 Windows 微软商店进程直连规则：`WinStore.App.exe` 和 `StoreDesktopExtension.exe`。它们位于 Bettbox 使用的 `sing-mix_origin` 文件中，更新上方 Raw 链接即可获取。`RuntimeBroker.exe` 未添加，因为 Windows 的多个应用会共用该进程名。

## 自动 AI 更新

GitHub Actions 每周一检查 [上游 Origin 脚本](https://github.com/Sakyvo/sing-mix/blob/main/sing-mix_origin)。发现上游或候选脚本有变化时，会调用你配置的 AI 服务比较上游与当前脚本，生成择优更新候选；通过 JavaScript 语法检查，并确认 Cloudflare 规则及日志修复的 8 条精确代理规则仍保留。验证通过后，Action 会更新自动分支、创建或更新 PR，并自动合并到 `main`。如果 AI 调用或检查失败，流程会停止，不更改 `main`。

自动合并后，GitHub 上方 Raw 链接对应的 `main` 脚本会更新；Bettbox 中已保存的配置不会因此自动变化。要让新脚本在 Bettbox 生效，还需要在 Bettbox 中手动点击“同步”，让它通过该链接重新拉取最新内容。

首次启用自动合并时，需先通过一次 PR 将此工作流更新合入 `main`；之后上游脚本更新才会由 Action 自动处理。

该工作流使用 OpenAI Chat Completions 兼容接口，所以可接入支持此接口的中转服务或 DeepSeek。先确认你的服务商提供兼容的 `/chat/completions` 路径、Bearer 密钥认证和 `choices[0].message.content` 响应格式。

首次使用前，在仓库 **Settings → Secrets and variables → Actions** 中配置：

- Repository secret `AI_API_KEY`：所选服务的 API 密钥。只保存到 GitHub Secret，不要写进文件或提交到仓库。
- Repository variable `AI_API_BASE_URL`：服务商给出的 HTTPS API 根地址，不带 `/chat/completions`。例如 DeepSeek 的根地址是 `https://api.deepseek.com`；中转服务请使用其文档提供的地址。
- Repository variable `AI_MODEL`：服务商支持的模型 ID。可填 DeepSeek 文档给出的模型名，或中转服务提供的模型 ID。

使用 DeepSeek 时，按 DeepSeek API 文档配置 Base URL、模型 ID 和对应 API Key；使用中转站时同样填写中转站提供的三项信息。GitHub Actions 会在检测到上游变更后将两份脚本发送给所配置的服务处理，因此应选用你信任的服务，并留意其数据和计费政策。API 用量由该 API 服务商计费，ChatGPT 订阅通常不包含 API 调用额度。

可在仓库 **Actions → AI review upstream sing-mix → Run workflow** 手动触发检查。定时任务使用 GitHub Actions；若 Actions 因仓库长期无活动被停用，需要在 Actions 页面重新启用。

### 审查 Pull Request

`.github/workflows/review-own-pull-requests.yml` 会审查所有目标为 `main` 的非草稿 PR，检查修改脚本的 JavaScript 语法，将 PR 差异发送给上面配置的 AI 服务，并在 PR 下留言。AI 明确给出 `APPROVE` 且工作流步骤成功时，只有 `bobvhfhh` 或 `github-actions[bot]` 创建的 PR 才会自动以 squash 方式合并。其他账号的 PR 也会收到 AI 审查，但由维护者决定是否合并。拒绝、接口错误或语法检查失败都会阻止自动合并。只有修改这份 AI 审查工作流本身的 PR 才需要人工合并，以防 PR 自己改写审查条件后自动放行。该流程只审查普通 PR，不让 AI 改写 PR 文件。

新增或修改此工作流后，必须先把包含它的 PR 手动合入 `main`。此后，目标为 `main` 的 PR 在新建、更新、重新打开或从草稿转为就绪时会触发审查。已经打开的 PR 不会因工作流刚合入而自动补跑；合入后可向该 PR 推送一次更新来触发，或关闭再重新打开。
