---
name: guwenzi-read
description: Read and explain ancient-character papers and excavated-text material with separate text and glyph evidence. Use with guwenzi-remote for atomic tools; do not force a fixed OCR pipeline or use this skill for glyph composition and paper typesetting.
---

# 古文字论文阅读与证据整理

先加载 `guwenzi-remote` 了解工具契约。阅读过程由 Agent 判断下一步，不要把“上传—整页准备—批量切槽—逐槽下载”当作默认流水线。

## 两类文献

### 文字型 PDF

保留并优先阅读原生文字层、字体、PDF path、内嵌图片和坐标。普通文字不需要重新 OCR。遇到无编码 path、来源编码异常、图片字或文字层与页面不一致时，再把相关局部转成图像，调用 `crop`、`glyph_search`、`glyph_register` 或 `glyph_compose`。

原文、作者释读、Agent 修订和字形身份必须分开记录；不要用释读覆盖原文字层。

### 图片型 PDF / 图片 / Office 页面

先拿到页面图，再按需调用 `layout_analyze` 的 `regions` 模式。YOLO只负责提出文字、图、表、图注等区域；图表按顺序保存，不强迫识别。Agent查看放大的文字区域后：

1. 能读出的文字直接输出整块文字；
2. 不确定某个字才选择局部框并 `crop`；
3. 仍不确定才 `glyph_search`；
4. 检索未命中时，回到原句、括注、作者明说的字名和清楚构形，并查正式 Unicode/GlyphWiki；检索失败不等于无编码；
5. 确认没有合适编码时才 `glyph_register`，需要可排印的规整打印体再 `glyph_compose`；
6. 最后用 `transcribe` 保存完整文字区域及结构化字形引用。

不要默认生成所有字符槽，也不要把槽位边界当成字形真值。

## 定位、投影与最终裁框

- 最终单字 bbox 由 Agent 查看完整行、邻字、括注和部件后提出，再用 `crop` 按原图像素执行；裁切工具本身不识字。
- 投影适合已经纠偏、方向一致、行列和字距稳定、有连续白带的打印正文。优先用于分栏、分行和等宽字格的候选槽位。
- 脚注号、上下标、夹注、标点和比例字体只能把投影当建议；须合并孤立点与分离部件，并参考邻字间距。
- 拓片、摹本、楚简照片、粘连污损字形不让投影决定单字边界。推荐流程是：版面区域 → 纠偏 → 行列投影 → 字槽候选 → 部件合并 → 视觉复核 → 原像素裁切。

## 证据纪律

- `asset_id`是文件/图像字节身份；`span_id`是位置建议；候选`c1`是一次搜索内的临时编号；`gwz:`是具体字形资产身份。
- Unicode、Seal、PUA、字体 GID、字形身份和释读不是同一层信息。
- 不做 NFC/NFKC、繁简转换或静默码位替换。
- 原图和放大图都保留；放大图不增加位图信息。
- 无法确认就写“未解/待核”，不要用上下文强猜。
- YOLO类别、投影边界、OCR输出、DINO排序和模型选择都必须标记其证据状态。
- 但“保守”不等于忽略直接文字证据：原句明确写出“释为某字”且原排形相符时，可据两者确认；同时保留失败检索，说明相似度没有识别成功。

## 解读写作

每一条解读区分：

1. 页面实际可见内容；
2. 论文作者的释读和论证；
3. Agent 根据字形、文例、文献和候选比较作出的推断。

HTML/PDF解读应附原页图、相关原字形图、页码、block/asset 引用和不确定项，并明确 PDF 物理页与书内页码及跨页断句。不要只输出一串现代 Unicode 字符而丢掉原始字形。

规整打印体还应尽量提供可复制的文字层：已确认字符直接使用正式 Unicode；真正无编码的形才用 KAGE 重建和字体局部 PUA，并随附字体与 mapping。正文可显示字体字形，点击回看论文原排图。甲骨、金文、楚简原形仍用原图，不以现代楷形或重建字体替代证据。

`transcribe` 只是保存 Agent 已经完成的阅读结果，不是 OCR，也不会自行判断哪些字需要检索。
