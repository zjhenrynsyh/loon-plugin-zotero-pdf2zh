# Loon 分流插件 · Zotero PDF2zh

给 [Zotero PDF2zh](https://github.com/guaguastandup/zotero-pdf2zh)（Zotero 的 PDF 中文翻译插件）用的 Loon 分流插件。

只做一件事：**让翻译接口走代理，让模型和依赖下载留在直连**。

## 为什么需要它

Zotero PDF2zh 的翻译引擎 `pdf2zh_next` 内置两个免费翻译端点：

```
https://api1.pdf2zh-next.com/chatproxy
https://api2.pdf2zh-next.com/chatproxy
```

这两个域名都在境外。直连时 TLS 握手会被中途掐断：

```
RemoteProtocolError: Server disconnected without sending a response
ConnectError: [SSL: UNEXPECTED_EOF_WHILE_READING] EOF occurred in violation of protocol
curl: (35) LibreSSL SSL_connect: SSL_ERROR_SYSCALL
```

`babeldoc` 的重试耗尽后会 `try fallback` —— **保留英文原文**。于是整篇翻完，你会发现一大堆段落根本没译，看起来像「插件漏翻」，实际是请求失败留下的空洞。

把这两个域名固定走代理，这一类失败就消失了。

同时，翻译引擎运行时要下载的模型与依赖（modelscope 的 doclayout ONNX、CMap 字体、PyPI 源）都在国内，走代理反而更慢更费流量，所以明确写 `DIRECT`。

## 规则一览

| 域名 | 策略 | 原因 |
|---|---|---|
| `pdf2zh-next.com` | 代理 | 翻译接口，直连 TLS 被掐 |
| `pdf2zh.com` | 代理 | 项目站点/接口备用域名 |
| `github.com` | 代理 | 更新检查 |
| `githubusercontent.com` | 代理 | 原始文件直连不通 |
| `githubassets.com` | 代理 | GitHub 静态资源 |
| `modelscope.cn` / `modelscope.com` | 直连 | doclayout 模型、CMap 字体 |
| `aliyuncs.com` | 直连 | 模型/依赖 CDN |
| `pypi.org` / `pythonhosted.org` | 直连 | 安装翻译环境 |
| `mirrors.ustc.edu.cn` / `mirrors.aliyun.com` / `tuna.tsinghua.edu.cn` | 直连 | PyPI 镜像 |

> 规则只写域名，不写 IP。Loon 的 fake-ip 模式下域名会被解析成虚拟地址（如 `198.0.9.x`），按 IP 段写规则不会命中。

## 安装

### macOS（Loon for Mac）

1. 下载本仓库的 `Zotero_PDF2zh.plugin`
2. Loon → 配置 → 插件 → 添加 → 选择本地文件
3. 启用插件，重载配置

也可以直接把文件放进 iCloud 的 Loon 目录（`iCloud 云盘 › Loon › Plugins`），在 App 内添加。

### iOS（Loon）

把 `Zotero_PDF2zh.plugin` 存到「文件」App，然后在 Loon → 配置 → 插件 → 添加 → 从文件导入。

## 需要改策略组名

插件里的策略名是 **`兜底后备策略`**，对应我这边配置的 `select` 策略组。如果你的策略组叫别的名字（`PROXY`、`节点选择`、`手动切换`……），把插件里的 `兜底后备策略` 全部替换成你自己的组名即可，其余不用动。

> 注意：配置文件里存在有效策略组时，Loon 内置的 `PROXY` 策略组不可用，必须填配置里真实存在的组名。

## 实测数据（同一台机器，各 10 次请求）

| 域名 | 加插件前 | 说明 |
|---|---|---|
| `api1.pdf2zh-next.com` | 5/10 成功 | 失败全是 TLS 握手被掐 |
| `api2.pdf2zh-next.com` | 3/10 成功 | 同上，且公共 DNS 被污染 |
| `www.modelscope.cn` | 302 正常 | 直连可用 |
| `raw.githubusercontent.com` | 0/3 | 直连完全不通 |

## 说明

- 插件只含 `[Rule]` 段，不需要 MITM，不修改任何请求内容。
- 如果加上规则后接口依然频繁失败（成功率没有改善），那问题在免费接口自身的限流，不在分流——建议换用带 Key 的翻译服务（DeepSeek / SiliconFlow / OpenAI 兼容端点）。

## 许可

MIT
