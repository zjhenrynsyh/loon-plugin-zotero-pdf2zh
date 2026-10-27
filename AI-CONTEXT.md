# AI-CONTEXT — Loon 分流插件（Zotero PDF2zh）

给 AI 助手看的项目上下文。人类读者请先看 `README.md`。

## 项目定位

一个**纯规则型**的 Loon 插件仓库，唯一目的：让 Zotero PDF2zh 的翻译请求走代理、模型与依赖走直连。

建立背景（2026-10-27）：用户用 Zotero PDF2zh 4.1.7 翻译一篇 15 页论文，结果双语版里大量段落仍是英文。
排查结论是 `pdf2zh_next` 的免费翻译端点 `api1/api2.pdf2zh-next.com` 在直连时 TLS 握手被中断，
`babeldoc` 重试耗尽后 `try fallback` 保留原文，于是表现为「漏翻」。分流规则是这一环的修复。

## 文件说明

| 文件 | 作用 |
|---|---|
| `Zotero_PDF2zh.plugin` | 插件本体，只有 `[Rule]` 段 |
| `README.md` | 面向用户：为什么需要、规则表、安装、策略组名替换说明、实测数据 |
| `CHANGELOG.md` | 按版本记录变更，**每次改动必须同步** |
| `LICENSE` | MIT |

## 关键事实（改之前先读）

- 翻译端点定义在 `pdf2zh_next/translator/translator_impl/siliconflowfree.py` 的 `AVAILABLE_SERVER_ENDPOINTS`：
  `https://api1.pdf2zh-next.com/chatproxy`、`https://api2.pdf2zh-next.com/chatproxy`。
  上游若新增端点，需要同步加 `DOMAIN-SUFFIX`。
- 端点全部**按域名访问，没有硬编码 IP**。
- 用户的 Loon 是 **TUN 模式 + fake-ip**：域名会被解析成虚拟地址（实测 `198.0.9.140`），
  **所以规则只能写域名，写 IP 段无效**。不要"优化"成 IP 规则。
- 插件里的策略名 `兜底后备策略` 是用户 `mac.lcf` 里真实存在的 `select` 策略组；
  其配置同时有 `FINAL,兜底后备策略` 与远程 `GEOIP,CN,DIRECT` 规则。
  存在有效策略组时 Loon 内置的 `PROXY` 不可用，不要改成 `PROXY`。
- 直连与代理的分工依据（实测）：`modelscope.cn` 302 正常（直连），`raw.githubusercontent.com` 直连全挂（代理）。

## AI 修改与发版标准流程

1. 改 `Zotero_PDF2zh.plugin`
2. 同步 `README.md`（规则表、数据）与 `CHANGELOG.md`（新增版本条目）
3. 提交前自检（用 Grep 工具，不用裸 shell grep）：
   - 陈旧词：`未生效|待构建|无法编译|暂不可用|待对齐|不会触发`
   - 占位符：`你的用户名|<版本>|旧仓库地址|旧机器名`
   - 插件文件里不应出现示例域名、注释里的旧策略名
4. 提交并打 tag：`git tag vX.Y.Z && git push origin main --tags`
5. **必须发 Release**（不能只 push 代码），把 `Zotero_PDF2zh.plugin` 作为资产上传，
   Release body 复制本版本 CHANGELOG 段落
6. 上传后 `curl -L` 回读资产大小，与本地文件比对，确认不是空文件

## 禁止事项

- 不要加 `[Rewrite]` / `[Script]` / `[Mitm]` 段——本插件不碰请求内容，MITM 会引入证书信任面。
- 不要把域名规则改成 IP / IP-CIDR。
- 不要在 README 里写死用户个人的节点名、订阅地址等私密信息。
- 不要为"看起来更全"而加入与本链路无关的第三方域名（如流媒体、广告域名）。
