---
name: guwenzi-read
description: 提取并解读古文字论文、扫描资料、Word或PPT，保存原始字形、图表、位置与可重现字形码。用于资料提取、训释梳理及笔画或异体变化比较；不负责新造字与论文排版。
---

# 古文字资料提取与解读

使用公开的 `guwenzi-tools` CLI 串联提取，无需启动MCP或重写PDF解析器、OCR、检索器。

已有远程工具服务时，优先使用安装的 `guwenzi-tools`。先运行 `guwenzi-tools agent guide --plain` 读取工具契约；有 Open Glyph 网站账号则 `guwenzi-tools login` 一次即可，凭据可随时注销；自部署服务由用户提供地址与凭据并用 `configure`。`document open` / `page prepare` 取页与原图，`layout_analyze` 可独立请求YOLO区域和投影候选，`crop` / `glyph search` / `glyph select` 处理不确定原形，外部多模态模型负责转写，再用 `transcribe` 回填与 `document export` 导出。

## 工具入口

在任意目录运行。CLI输出JSON和本地证据目录，不向模型提示词写入本机路径。

```sh
guwenzi-tools doctor --quick
guwenzi-tools document open paper.pdf
guwenzi-tools page prepare DOCUMENT_ID 1 --output work/page-1 --detach
guwenzi-tools status JOB_ID --wait --output work/page-1
```

长任务使用`--detach`并用`status JOB_ID --wait --output DIR`收取。结果有`next_offset`时分页读取直到为空；不要在提交结果不确定时盲目重复POST。

PDF逐页可以走原生文字/路径提取，也可以把图片页交给YOLO区域分析与投影候选。图表单独保存，原始路径不以SVG替代；外部多模态模型决定是否查看、裁切、检索或回填。具体参数以`guwenzi-tools agent guide --plain`和服务端`tools --server`返回的schema为准。

## 原形与标识

- `asset_id`是具体图像或文件字节；`span_id`是本行位置；候选`c1`仅本次请求有效。
- `gwz:`是可重复输入的具体字形资产身份，不等于释读完成。
- 字符身份、字体借码、具体形体和作者释读分开记录。不要NFC/NFKC归一化或繁简替换。
- 未解字、候选和原始Unicode/Seal/PUA均保留，不能因为模型给出合法码位就认定笔画一致。

```sh
guwenzi-tools asset 'asset:从result取得的哈希' --output work/asset.png
guwenzi-tools asset 'asset:从result取得的哈希' --view --output work/asset-view.png
guwenzi-tools glyph resolve 'gwz:从result取得的永久码'
```

命令输出路径和元数据，再用看图工具打开。原生矢量另有直接从源PDF生成的高清细节图；位图放大没有增加原始信息。不要向上下文倾倒整库JSONL或图片base64。

## 解读与比较

用有序正文建立论证结构，再查看具体讨论对象和图表。引用记录页码、block_id、原图或字形码。区分作者观点、字形观察和模型推测。

`source_text_unverified_encoding`是原文字层；`ocr_unverified`是视觉初读；`model_selected_unverified`是候选选择；`original_asset_unresolved`只是原形记录。`complete`表示程序完成，不能写成全篇百分百识别。

括号可能标隶定、通假、读法或解释，不能直接当成图中文字的Unicode标签。同形在不同语境中也可能有不同释读，不用多数票统一原文。

输出应可追溯论文信息、论证、具体字形变化和未确定部分。阅读顺序异常和疑似漏行回查整页原图；不要求用户先知道字体或码位。
