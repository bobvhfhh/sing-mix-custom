像 singbox 一样的丝滑的 Clash 覆写脚本

---

Origin（国家/地区额外分组，Cloudflare 验证规则走 main 代理组）
```text
https://raw.githubusercontent.com/bobvhfhh/sing-mix-custom/refs/heads/main/sing-mix_origin
```

基于 [Sakyvo/sing-mix](https://github.com/Sakyvo/sing-mix) 的 Origin 脚本，仅将 Cloudflare 验证规则的策略从 `DIRECT` 改为 `main`，其余分流逻辑保持原样。
