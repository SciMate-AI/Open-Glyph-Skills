# Open Glyph Skills

这是 Open Glyph（开放古文字）的可复用 Agent Skills 集合，供能够读取 `SKILL.md` 的 Agent 使用。

## 三个 Skill

- [`guwenzi-remote`](guwenzi-remote/SKILL.md)：最基础、最重要的工具调用 Skill。说明如何使用 `guwenzi-tools` 登录、提交文档、调用 YOLO 版面分析、投影裁切、DINO 字形检索、KAGE 造字、转写和导出。它强调每次读取结果后由 Agent 自己决定下一步，不要求固定流水线。
- [`guwenzi-read`](guwenzi-read/SKILL.md)：古文字论文、扫描资料、PDF、Word、PPT 的证据提取、阅读、比较与释读记录。
- [`guwenzi-compose`](guwenzi-compose/SKILL.md)：古文字组字、KAGE 据图重建、字体映射和古文字论文撰写排版。

推荐使用顺序：先加载 `guwenzi-remote` 了解工具契约；需要阅读资料时再加载 `guwenzi-read`；需要造字或撰写排版论文时再加载 `guwenzi-compose`。

## 安装 CLI

工具调用 Skill 依赖公开的 `guwenzi-tools` 客户端：

```sh
python -m pip install -U guwenzi-tools
guwenzi-tools login
```

登录凭据由用户在 Open Glyph 网站中授权。Skill 不包含服务地址、Token、AccessKey 或任何用户数据。

## 安全与证据边界

本仓库只包含 Markdown 指令，不包含模型权重、论文、字库图片、用户文件、服务器 IP、EAS 地址、API Token、AccessKey、私有路径或部署配置。

Skill 要求 Agent：

- 保留原始文件、原始图片、路径程序、哈希和页面坐标；
- 不做 NFC/NFKC、繁简转换或静默码位替换；
- 将来源编码、具体字形、永久 `gwz:` 身份和作者释读分开记录；
- 将 YOLO 类别、投影槽位、OCR 输出和 DINO 相似度视为证据或候选，而不是自动确认；
- 不把 `c1`、`asset_id`、`span_id`、Unicode、Seal、PUA 和 `gwz:` 互相混用；
- 对未确认的字形保留原图和不确定状态，不编造释读。

## 许可证

本仓库中的 Skill 文本使用 MIT License。外部 CLI、模型、字体、KAGE 引擎、GlyphWiki 数据、论文和字库资产仍分别受其自身许可证约束；本仓库不重新授权这些外部材料。
