---
name: guwenzi-remote
description: Use guwenzi-tools as an agent-controlled set of atomic document, image, glyph-search, and glyph-composition tools. Read pages, preserve evidence, output text, and call refinement tools only when the visual model is uncertain.
---

# Guwenzi remote tools: agent decides the next action

This is an atomic-tool contract, not a fixed OCR pipeline. The remote service does not contain a multimodal language model and does not decide which glyph needs research. The calling agent must inspect returned images, write the text it can read, and request more evidence only when necessary.

## Authentication and safety

```sh
python -m pip install -U guwenzi-tools
guwenzi-tools login
guwenzi-tools whoami
guwenzi-tools doctor --quick
```

Self-hosted services use `guwenzi-tools configure --endpoint URL`; never place a shared token in a prompt, source file, HTML, or evidence package. `tools --server` returns the live JSON schemas.

Every output must preserve the distinction between original file/page/image bytes, `asset_id`, `span_id`, temporary candidate IDs such as `c1`, `gwz:` glyph identity, source-font codepoints and aliases, and an author's reading or the agent's interpretation.

Never apply NFC/NFKC, traditional/simplified conversion, silent codepoint substitution, or a guessed reading. Similarity is not a probability and a selected candidate is `model_selected_unverified` until checked.

## Atomic tool map

Choose tools from the current evidence; do not execute every row:

| Need | Tool/command | Result |
|---|---|---|
| Register a whole file | `document_open` / `document open` | Original bytes, hash, page count; no reading |
| Register a page image already available | `asset_import` | Page `asset_id` and enlarged view |
| See one image more clearly | `asset ASSET_ID --view` | Enlarged derivative, no new detail |
| Ask YOLO where regions are | `layout_analyze` with `analysis_mode=regions` | Text/figure/table/caption proposals and overlay; no OCR |
| Get optional projection lines/slots | `layout_analyze` with `regions_lines` or `split_line` | Suggestions only; use only when AI needs localization |
| Take a precise local crop | `crop ASSET_ID --box L T R B` | Original-pixel crop and provenance |
| Search a doubtful glyph | `glyph_search` / `glyph search` | Candidate images and neutral `cN` IDs |
| Commit a candidate choice | `glyph_select` | Binding and source identity, still unverified |
| Preserve a glyph with no match | `glyph_register` | `gwz:sha256:` identity for the original bytes |
| Reconstruct a printed bitmap | `glyph_compose` | Model-written KAGE derivatives, labelled reconstruction |
| Save the agent's reading | `transcribe` | Structured text and glyph references stored with page/block evidence |

`page_prepare` and `document prepare` are compatibility operations from an older combined workflow. Do not use them as the default reading strategy: they can run layout, projection and bulk slot creation before an agent has decided that any of those are needed. Prefer a page-image/native-evidence primitive exposed by the host, then call the atomic tools above. If a legacy host exposes only `page_prepare`, treat its result as unverified evidence, not as OCR or a finished transcription.

## Image-page decision pattern

For a scanned or image-only page:

1. Obtain the page image and let the agent inspect it at a readable scale.
2. Call YOLO layout analysis with `regions`, not all slots. Keep figure/table regions as ordered visual objects.
3. Inspect each proposed text region. If the agent can read a region, transcribe the whole region directly; do not download every slot.
4. If one character is uncertain, choose a crop box from the readable region and call `crop`.
5. Call glyph search only for that crop. Compare the actual candidate images, not just scores.
6. If a candidate is useful, select it as an unverified binding. If no candidate is defensible, register the original crop; if a printed form needs a usable character, write and validate a KAGE recipe with `glyph_compose`.
7. Submit the complete region text with structured glyph/image references through `transcribe`.

The agent, not the service, decides whether a character is known, uncertain, unencoded, or worth reconstructing. Do not turn a table into hundreds of forced OCR slots.

## Native/text PDF decision pattern

For a PDF with a text layer:

- preserve and read the native text as the source layer;
- preserve embedded images, fonts, paths, and page coordinates;
- use visual evidence to check encoding gaps or suspicious text;
- if an unencoded path/image character is unknown, crop or retrieve that individual form using the same atomic image tools;
- submit an external correction, reading, or glyph reference separately from the original source text.

Native extraction is not a reason to skip visual evidence, and visual retrieval is not a reason to overwrite the source text.

## `transcribe` semantics

`transcribe` is not an OCR command and does not invoke an AI model. It persists text that the calling agent has already read, with parts such as `text`, `binding_id`, `glyph_code`, or `asset_id`. Use it only after reading the relevant region and checking the evidence. Keep the source layer, the agent's transcription, and scholarly interpretation separate.

## Uncertainty and resumability

- Read the original page/region image before interpreting a candidate.
- A projection boundary is not proof that one crop is one character; correct it with `crop` when needed.
- Use `--detach` and `status JOB_ID --wait --output DIR` for long jobs.
- Never repeat an uncertain POST blindly. Keep the job receipt and inspect its status first.
- Do not place base64 images or entire catalogues in the model context; fetch only the evidence needed for the next decision.

## Minimum final record

A useful result contains the original source hash, ordered text/figure/table objects, page and block citations, original and derived image hashes, any search/selection evidence, unresolved glyphs, and a clear status for every claim. Completion of a tool job never means that the text or reading is correct.
