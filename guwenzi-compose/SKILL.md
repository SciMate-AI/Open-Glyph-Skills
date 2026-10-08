---
name: guwenzi-compose
description: Compose or reconstruct ancient-character glyphs and write evidence-based palaeographic papers with reproducible glyph assets, KAGE recipes, optional OpenType fonts, and verifiable PDF output. Do not use this skill for ordinary reading alone.
---

# 古文字组字、重建与论文排版

本 Skill 只在用户要求组字、无编码字形重建、字体/字形表或古文字论文写作排版时使用。阅读文献和按需检索先使用 `guwenzi-remote` 与 `guwenzi-read`，不要预先批量处理整页或所有槽位。

## 字形层必须独立保存

始终区分：原始图片/path、来源字体/码位、`gwz:`具体资产、KAGE重建程序、派生字体、作者释读和 Agent 推断。任何派生 SVG、PNG、OTF 或现代字都不能替代原件。不要做 NFC/NFKC、繁简转换或静默码位替换。

## 何时造字

先判字符身份，再决定是否造字：

1. 原句、括注或清楚字形能确认正式 Unicode：直接使用该码位，需要统一排印风格时渲染对应现代 KAGE；原排图仍作为证据。
2. 有原始 PDF path 或字体轮廓：优先保留原始曲线和程序，字体只是派生排印结果。
3. 确实无编码且只有规整打印体位图：观察具体部件，写出自包含 KAGE 配方，调用远程 `glyph_compose` 或明确安装的 `guwenzi-glyph`，查看原图和渲染图后修订。输出标记“据图重建 / model reconstruction”。
4. 拓本、手写、器物照片、残损或来源不确定：保留图片，不强行重建。

不要因为 OCR 或图像检索没命中就宣布“无编码”；相似检索失败不是字符身份判断。构形清楚的打印体优先用 KAGE，不把扫描图二值描边当作主要排印字体，以免把锯齿、墨晕和扫描噪声固化进字形。

## 远程 KAGE 工具

```sh
guwenzi-tools call glyph_compose --json @recipe.json --output work/reconstruction.json
```

配方应包含原图哈希、`source_kind`、构造理由、root 和完整组件表；保留精确 `@版本`，不剥除版本或静默替换组件。渲染成功不代表恢复了原字体，更不等于获得了 Unicode 释读。

## 本地伴随工具

`guwenzi-tools` 是轻量远程客户端，不自动安装 Node、KAGE 引擎或 GlyphWiki 数据。若用户明确安装了 `guwenzi-glyph`，再以其 `--help` 为准调用 `lookup`、`parts`、`ids`、`render`、`compose` 和 `build-font`。否则优先使用远程 `glyph_compose`，不要假设本机存在私有路径。

## 论文论证

每个新释读至少分开讨论：字形依据、同文/文例依据、文献旁证和反例检查。正文必须能回到页码、原图、字形资产和出处。括注释读不能替代原形；相似字形不能自动证明同字同音。

## 字体编码

- 有正式 Unicode 的字继续映射正式码位；不要为了保存论文中的每一个排形而全部改成 PUA。
- 只有无正式码位、但确需输入的字形/部件落入字体局部 PUA。PUA 只是该字体 `cmap` 中的调用地址，不是官方字符身份。
- 同一字符的论文原排异体若需精确区分，可另保留字形实例和原图；语义文本层仍优先使用正式 Unicode。
- `build-font` 若默认把所有输入分配到 PUA，最终组装时要把已确认字符恢复到正式码位，并在 mapping 中只留下真正无编码项的 PUA。

## 排版交付

交付原始图/path、KAGE recipe、组件版本、逐字 SVG、可选 OTF、字体 SHA、Unicode/PUA 映射表、原图—字体并排校样、manifest、可编译源码和最终 PDF。PUA 只在对应字体和 mapping 中有效；没有原始轮廓时保留图片作为研究证据。实际打开或打印产物检查字形非空、码位可复制、字体已嵌入。编译成功和视觉相似都不等于学术确认。
