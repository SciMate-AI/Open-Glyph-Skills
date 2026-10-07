---
name: guwenzi-compose
description: 组合古文字字形(部件拼合、IDS 隶定造字、为无编码字形造字并嵌入 PDF)并撰写框架严密、思路清晰、可核对的古文字学论文 PDF。当用户要求"造字/组字/拼字""写一篇古文字论文""把这个思路写成论文""做一个字形表""排版一篇出土文献研究文章"时使用。产出包括原始字形证据、构造代码及可选 OTF 字体、论证框架、以及可编译的论文源码与 PDF。不要用于只需阅读论文的任务(那用 guwenzi-read)。
---

# guwenzi-compose:组字与撰写古文字论文

## 字形证据与排印

字形细节是论据。保留原图、原始字库身份、PDF 路径程序及出处；派生 SVG、重建代码和字体都不能取代原件。位图可放大查看，但不因此增加原始信息。不要用括注释读替换原字形，也不要把 gwz 永久身份等同于 Unicode 字符。

按实际来源选择：

1. 已有原始字体或 PDF 轮廓：优先沿用。远程页面证据包可用 `guwenzi-tools font build --packet work/page-1 --output work/font` 导出 OTF、字体 SHA 和 PUA/gwz 映射。保留原始 `.path` 字节。
2. 只有打印体位图：观察具体笔画，写出自包含 KAGE 配方，调用远程 `glyph_compose` 或明确安装的 `guwenzi-glyph`，查看原图和渲染图、逐笔修订。产物标为“据图重建”，保留所有修订及原图，不宣称恢复原字体。
3. 拓本、手写、器物照片或分类不确定：保留原图，用图片嵌入正文及字形表。被印进论文的拓本仍是拓本，不因此变成可重建的打印体。

GlyphWiki 包含多类变体和专门字形；逐条核验本地 dump 和来源。既不能断言它覆盖全部古文字，也不能断言没有甲骨、金文相关字形。

## 复用已有造字工具

```sh
guwenzi-glyph lookup 則
guwenzi-glyph parts 則
guwenzi-glyph ids "⿰木目"
guwenzi-glyph render 則 --curve -o work/glyphs/
```

`ids` 查的是 dump 中已有组合；查不到不代表 KAGE 不能画。旧 `render/ids` 可能用 newest-only 数据代替带版本的部件，它们不是历史字形的精确复原。研究用的构造必须保存实际采用的完整组件代码和哈希；不得把剥去 `@版本` 后的组件冒称原版本。99 型部件名在字段下标 7。

打印体位图的配方入口：

```sh
guwenzi-glyph compose recipe.json --source original.png --output work/revision-1 --font
```

`guwenzi-glyph` 是可选的本地伴随工具；`guwenzi-tools` 的远程服务也提供 `glyph_compose`。安装 `guwenzi-tools` 不会自动安装本地 Node、KAGE 引擎或字库。使用本地工具时以 `guwenzi-glyph --help` 为准；使用远程工具时以 `guwenzi-tools tools --server` 返回的 schema 为准。

配方使用 `guwenzi.kage-construction.v1`，包含 source_kind、source_sha256、rationale、root、table；table 是完整的 KAGE 部件闭包。根名和 `@版本` 精确匹配；缺部件、循环或空输出报错。保留原始输入 SHA，不能拿放大图的哈希冒充原图的永久码。每次修改使用新输出目录，配方的 previous_revision 可记录上一版哈希。

输出 original、recipe.json、root.kage、components.json、render.svg、1024 像素 preview.png、comparison.html 和 reconstruction.json；`--font` 另导出 OTF 及映射。SVG 仅为渲染中间产物，KAGE 原始代码才是构造程序。外部模型决定如何画，CLI 不会自动“识别位图”。

逐笔比较笔数、连接、交叉、位置、方向、粗细和笔端。局部不明则保留不确定性，不自行补全。合成完或像素相似不等于研究者认可，也不能反推 Unicode 释读。

已有 KAGE SVG 也可导出：

```sh
guwenzi-glyph build-font 'work/glyphs/*.svg' -o work/custom.otf
```

OTF/CFF 保留曲线，多次独立填充先做几何并集。旧 `.ttf` 路径会采样曲线，仅留作兼容，不用于精细比较。PDF 路径导出直接读原始代码，不先转 SVG。

PUA 属于具体字体；文档同时交付字体、字体哈希和映射。`guwenzi-tools font encode text.txt --map work/font/glyph-map.json --output work/typeset.txt` 将 `⟦gwz:…⟧` 转为字体内码位。永久身份仍为 gwz，不能裸传 PUA 后丢掉字体。HTML reading.html 则可以直接用高清图片行内显示，无需字体。

KAGE 引擎及外部字库的许可证独立于客户端 MIT；引擎单独安装，不打进轻量客户端包。

## 第二阶段:论文框架

### 2.1 古文字论文的标准结构

按出土文献研究的实际体例,不是通用学术模板:

```
标题
作者 / 单位
提要         ← 200 字内,点明新见
关键词       ← 清華簡 / 參不韋 / 出土文獻 / 考釋

一、問題的提出
    前人研究综述(谁说过什么,原文引)
    本文要解决的问题

二、字形與釋讀
    (一) 某字
        整理者原釋 → 引原文
        本文新釋 → 字形依据(必须给字形图)
        文献旁证 → 传世典籍/其他出土材料
    
三、詞義與文例
    简号标注:簡 100 / 簡 45
    释文体例:
        參不韋曰:啟(啟),隹(唯)昔方有洪(洪)
                   ↑原字形   ↑今字

四、結論

附錄:字形表
參考文獻
```

### 2.2 必须遵守的体例规范

| 项 | 规范 | 例 |
|---|---|---|
| 简号 | `簡` + 阿拉伯数字 | 簡 100 |
| 隶定 | 原字形后括号注今字 | 隹(唯) |
| 通假 | 用括号 | 秉悳(德) |
| 缺文 | 用 `□` 或 `【】` | 【患】 |
| 韵读 | 上标数字 | 〔1〕 |
| 引书 | 《》 | 《清華大學藏戰國竹簡(拾貳)》 |

### 2.3 论证的严密性要求

**每一条释读必须给出四要素**:

1. **字形依据** —— 字形图 + 与何字形体相近
2. **文例依据** —— 在简文其他位置是否有同样用法
3. **文献旁证** —— 传世典籍或其他出土材料
4. **反例检查** —— 为什么不是其他可能

**禁止**:

- 只给结论不给字形图
- 用"疑""或"含糊掩盖没把握(有把握度就明说)
- 引用前人观点不给出处

## 第三阶段:排版与编译

### 3.1 工具链

```bash
# 首选 XeLaTeX(中文 + fontspec 最省事)
xelatex paper.tex

# 或 Typst(更快)
typst compile paper.typ
```

### 3.2 用自定义字体排古文字

```latex
\usepackage{fontspec}
\newfontfamily\guwenzi{custom.otf}[Path=./]
% PUA 用法的字,用十六进制写:
\newcommand{\gw}[1]{{\guwenzi\char"#1}}
% 用法:\gw{E000}
```

### 3.3 字形图保持来源形式

有原始轮廓时可用字体或矢量图。只有位图、拓本或照片时保留原始图片，并给出出处、原始尺寸及放大查看入口。例如：

```latex
\usepackage{graphicx}
\includegraphics[height=1.5em]{glyphs/p03_g001.png}
```

不要为了矢量化而抹掉残损、补画笔画或替换原件。重建图需单独标注。

## 交付清单

一篇合格的产出必须包含:

- [ ] `paper.tex` / `paper.typ` —— 可编译源码
- [ ] `paper.pdf` —— 编译产物
- [ ] `glyphs/` —— 每个字形的原始图片/路径程序、可选构造代码及派生图，命名含出处
- [ ] `custom.otf` 和映射表 —— 造字字体(若有)
- [ ] `pua_map.json` —— PUA 码位映射表
- [ ] `manifest.json` —— 字形 ↔ 正文位置 ↔ 释读依据
- [ ] `README.md` —— 复现步骤

## 硬性纪律

1. **字形必须放大保存。** 研究者要看细节变化,小图没有价值。
2. **每个字形要有出处。** 哪个简号、哪个位置,可追溯。
3. **区分无损与有损。** 矢量可无限放大;位图放大不增加信息,要标注原始尺寸。
4. **区分原件与重建。** 据图重建可用于排印，必须标注，不能冒充原始笔画。
5. **编译不过不算完成。** 必须实际跑通编译并检查产物。
6. **核验实际覆盖。** 字库匹配须有可追溯来源，找不到时保留原图。

## 工具位置

在guwenzi项目根目录使用仓库内入口（项目位置可配置，不依赖任何个人目录）：

```bash
bin/guwenzi-glyph info          # 引擎与数据状态
bin/guwenzi-glyph setup         # 首次使用下载 GlyphWiki dump，约110MB
bin/guwenzi-extract --help      # 本地提取工具组
```

均失败时先确认当前目录是项目根，再查 `tools/guwenzi-glyph/README.md`。
