# stash_overwrite_tsopt

一个面向 Stash iOS/macOS 的覆写配置（`.stoverride`），在机场 Clash 订阅的基础上提供完整分流规则，并针对 Tailscale 访问做了优化。

## 订阅地址

在 Stash 中通过 URL 添加覆写配置：

```text
https://raw.githubusercontent.com/CakewthinSUm/stash_overwrite_tsopt/main/Stash-main.stoverride
```

[直接打开 Raw 配置](https://raw.githubusercontent.com/CakewthinSUm/stash_overwrite_tsopt/main/Stash-main.stoverride)

## 前置条件

- **Stash 5.1 beta 或更高版本**：地区策略组使用 `smart` 类型，5.0 及以下版本会加载失败。
- **一份机场 Clash 订阅作为基础配置**：本文件是覆写（overwrite），不是完整配置；节点来自基础订阅，覆写只负责 Tailscale 节点、策略组和分流规则。
- **Tailscale 节点认证**：首次导入后，在 Stash 的节点菜单中打开 Tailscale 页面完成登录（配置文件中未写 `auth-key`）。

## 功能概览

- **Tailscale 直连**：Tailnet 域名（`ts.net`）、IPv4（`100.64.0.0/10`）和 IPv6（`fd7a:115c:a1e0::/48`）走专用的 `Tailscale访问` 组；节点配置了 `benchmark-url`，可在代理列表中观察直连延迟（约 20–40 ms）与 DERP 中继（100 ms+）。策略组图标使用 Tailscale 官方 logo。
- **地区智能选路**：香港、台湾、新加坡、日本、美国、韩国六个 `smart` 组按真实流量自动选节点，另有 `其他地区` 兜底；`自动选择` 采用反向筛选，自动纳入所有正常地区节点。
- **应用分流**：主体使用 Stash 内置 GEOSITE（随内核更新），覆盖 Telegram、Twitter/Facebook/Instagram、GitHub、OpenAI/Claude/Gemini（合并到 `AI服务`，默认新加坡）、YouTube、Netflix、Disney+、HBO、PrimeVideo、Apple TV+、Spotify、Bahamut（默认台湾）、PayPal、Microsoft、Google、TikTok、Steam、Emby、游戏平台、测速工具、国外媒体、国外电商等。
- **Steam / 游戏下载直连**：Steam 下载 CDN（`Steam_CDN.list`）优先于 `GEOSITE,steam` 命中，避免国区下载走代理；另有游戏平台下载 CDN / Tracker 直连。
- **Apple / APNs**：`Apple.list` 提供 GEOSITE 无法覆盖的 USER-AGENT 和 IP 段规则，走 `苹果服务` 组（默认 DIRECT）；APNs 域名与 Apple 官方推送 IP 段使用 dler-io 上游规则集，另有 TCP 5223 端口兜底，走 `苹果推送` 组。
- **统一策略组模板**：20 个应用组共用一份成员列表（YAML 锚点），统一为 `手动选择 → 香港 → 美国 → 新加坡 → 日本 → 台湾 → 自动选择 → DIRECT`；AI服务和Bahamut因特殊默认单独定义。

## 覆写合并方式

本配置对 `rule-providers`、`proxy-groups`、`rules` 使用 `#!replace`，即整段替换基础订阅的对应内容，最终结果完全由本文件决定，不受订阅自带规则集和策略组影响。

`proxies` 不加 `#!replace`，仅把 Tailscale 节点插入基础订阅的节点列表，保留机场节点。

## 规则来源

| 用途 | 来源 | 更新方式 |
|---|---|---|
| 应用分流（主体） | Stash 内置 GEOSITE（v2fly/domain-list-community 维护） | 随 Stash 更新 |
| Apple USER-AGENT / IP 补充 | [LingJingMaster/Shadowrocket-Rules](https://github.com/LingJingMaster/Shadowrocket-Rules) `Apple.list` | `main`，每日刷新 |
| APNs 推送域名与 IP 段 | [dler-io/Rules](https://github.com/dler-io/Rules) `Apple Push.yaml` | `main`，每日刷新 |
| Steam CDN 直连 | [Aethersailor/Custom_OpenClash_Rules](https://github.com/Aethersailor/Custom_OpenClash_Rules) `Steam_CDN.list` | `main`，每日刷新 |

## 导入后检查清单

1. 策略组中能看到 35 个组，`自动选择` 和六个地区组有节点成员。
2. Tailscale 节点状态正常，延迟约 20–40 ms；若显示 100 ms+，多半走了 DERP 中继。
3. 在 Stash「连接」页访问 Youtube / GitHub / Claude / Steam，确认命中的规则和策略组符合预期。
4. APNs 推送到达正常；如需排查，可在「连接」页查看 `苹果推送` 组的命中情况。

## 已知限制

- `DST-PORT,3478,DIRECT` 与 `DST-PORT,5223,苹果推送` 是全局端口规则，可能影响同端口的其他连接。
- APNs 推送依赖上游 dler-io 规则集，其 `apple.com.edgekey.net` 等 Akamai 域名规则仍可能误伤少量非推送流量。
- Stash iOS 3.6+ 默认对 Tailnet 域名和 100.64/10 自动路由，优先级高于配置文件规则；`Tailscale访问` 组选 DIRECT 在低版本才完全生效。
- 覆写的 `name` / `desc` / `icon` 仅用于界面展示，不参与配置合并，不会改变基础订阅的名称。
