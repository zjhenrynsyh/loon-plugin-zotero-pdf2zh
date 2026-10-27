# CHANGELOG

## v1.0.0 — 2026-10-27

首个版本。

- 新增 `Zotero_PDF2zh.plugin`：为 Zotero PDF2zh 翻译链路分流
  - `pdf2zh-next.com` / `pdf2zh.com` → 代理（翻译接口，直连 TLS 握手被掐）
  - `github.com` / `githubusercontent.com` / `githubassets.com` → 代理（更新检查、原始资源）
  - `modelscope.cn` / `modelscope.com` / `aliyuncs.com` → 直连（doclayout 模型、CMap 字体）
  - `pypi.org` / `pythonhosted.org` / USTC、阿里云、清华镜像 → 直连（安装翻译环境）
- 仅使用 `[Rule]` 段，无 MITM、无脚本、不改写请求。
- 附实测数据：`api1` 5/10、`api2` 3/10 成功率，失败均为 TLS 握手阶段被中断。
