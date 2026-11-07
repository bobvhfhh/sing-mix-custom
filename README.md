像 singbox 一样的丝滑的 Clash 覆写脚本

---

Origin（国家/地区额外分组，Cloudflare 验证规则走 main 代理组）
```text
https://raw.githubusercontent.com/bobvhfhh/sing-mix-custom/refs/heads/main/sing-mix_origin
```

基于 [Sakyvo/sing-mix](https://github.com/Sakyvo/sing-mix) 的 Origin 脚本，仅将 Cloudflare 验证规则的策略从 `DIRECT` 改为 `main`，其余分流逻辑保持原样。

## 跟踪上游更新

GitHub Actions 每周一检查 [上游 Origin 脚本](https://github.com/Sakyvo/sing-mix/blob/main/sing-mix_origin)。发现上游有变动时，会创建或更新一个 Pull Request，把上游原版放入 `upstream/sing-mix_origin` 供对比审查。这个 PR 不会覆盖上面的可用脚本。

审查 PR 时，将 `upstream/sing-mix_origin` 与 `sing-mix_origin` 做差异比较，由 AI 总结变更并挑选适合的部分；确认后再把选中的改动应用到 `sing-mix_origin` 并合并 PR。仓库不会自动择优或自动合并。

也可以在 GitHub 仓库的 Actions 页面手动运行 `Check upstream sing-mix`。首次使用前，请在仓库 Settings → Actions → General 中允许 workflow 创建 Pull Request；如权限未开，工作流会因 `pull-requests: write` 权限不足而失败。
