---
name: guwenzi-remote
description: Use the guwenzi-tools CLI to prepare historical-document evidence, inspect original glyph images, search DINO candidates, submit scoped transcriptions and export readable results through an Open Glyph website account or a self-hosted remote service. Use for PDF, image, Word/PPT extraction and palaeographic reading when this CLI is available.
---

# Guwenzi remote document reading

CLI stdout is always one JSON object `{"ok":true,"result":…}` or `{"ok":false,"error":{"code","message"}}`; progress/lifecycle events go to stderr. This package is a remote client: it never runs OCR/DINO/YOLO locally and never calls a multimodal provider. You need a terminal and an image-viewing tool.

## Connect

- If not configured, `whoami` fails with code `not_configured`. With an Open Glyph website account run `guwenzi-tools login` once: the user approves the shown code in their browser; no endpoint or token typing. `whoami` shows the account. `guwenzi-tools logout` revokes the credential server-side and removes the local profile; it can also be revoked in the website workspace.
- Self-hosted service instead: `guwenzi-tools configure --endpoint URL` and enter the token at the hidden prompt. Profiles coexist: `--profile NAME`, `config use NAME`, `config show` (always redacted). A stored credential is only sent to the exact endpoint it was saved for.
- `doctor --quick` checks health+ready; full `doctor` verifies remote resources. `tools` prints offline JSON Schemas; `tools --server` fetches live ones.

## Job lifecycle — read this before long operations

Every processing call (`page prepare`, `document open/prepare`, `crop`, `line split`, `glyph search/resolve`, `call`) submits a cloud job. The CLI writes `pending.json` (endpoint+job_id) into `--output DIR` **before** waiting, so work survives interruption.

- `--detach` returns `{job_id}` immediately; finish later with `status JOB_ID --wait --output SAME_DIR`. Always pass the same output directory: endpoint and job mismatches abort with code `resume_mismatch` instead of mixing results.
- `status JOB_ID` (no `--wait`) is a quick check; completed results without `--output` are returned as JSON only, without materializing images.
- Terminal killed or timeout: rerun the same `status … --wait --output` command. Exit code 130 means interrupted, not failed.
- Batch pages: `document prepare FILE --output work/book --pages all` keeps a resumable session; rerun with `--resume`; `--retry-failed` after fixing the cause. Large books on the small CPU service take minutes per page — prefer targeted pages.

## Errors — react by code, not by message

| code | meaning → action |
|---|---|
| `not_configured`, `login_required` | no credential → run `login` |
| `not_a_gateway` | endpoint is a raw service, not the website gateway → use login, or point commands at the right profile |
| `submission_unknown` | a POST may have reached the server → do NOT resubmit blindly; check job/result state first |
| `wait_interrupted` (exit 130) | user/time cut the wait → rerun status, the job still runs |
| 429 / `throttled` | per-user limit → back off and retry later |
| `slow_down` | login polling too fast → increase interval |

GET retries and explicit 429s are auto-retried with backoff; POSTs are never silently replayed.

## Capability map — decide after every result; no fixed order

| Action | Command | Input → Output |
|---|---|---|
| Provide a page image you already have | `upload page.png` then `call asset_import --json '{"input_key":"page.png"}'` | file → `asset_id` |
| Provide a whole PDF/image/Office | `document open FILE` | file → `document_id`+page count |
| One registered page → regions (YOLO) **without** creating a document | `call layout_analyze --json '{"asset_id":"…","analysis_mode":"regions"}'` | `asset_id` → regions, overlay, `boundaries_unverified` |
| One registered page → regions→lines→slots | `call layout_analyze --json '{"asset_id":"…","analysis_mode":"regions_lines"}'` | `asset_id` → regions + per-line `asset_id` + slot crops |
| Registered page inside a document (auto native/raster) | `page prepare DOC_ID N` | document + page → blocks, `asset_id` |
| Enlarge / fetch any image | `asset ASSET_ID [--view]` | `asset_id` → PNG file |
| Exact pixel crop (half-open `[l,t,r,b]`) | `crop ASSET_ID --box …` | `asset_id`+bbox → new `asset_id` |
| Projection-slot proposals from a line crop | `line split ASSET_ID` | `asset_id` → slots, `boundaries_verified:false` |
| DINO candidates for a crop | `glyph search ASSET_ID [--expand-search ID]` | crop `asset_id` → neutral `cN`+images |
| Bind candidate to permanent glyph identity | `glyph select SEARCH_ID cN` | search+candidate → `gwz:…` (unverified) |
| Look up a permanent code | `glyph resolve CODE` | `gwz:…` → codepoints/versions/aliases (not readings) |
| Save an unmatched original | `glyph register ASSET_ID` | `asset_id` → `gwz:sha256:…` |
| Model-written KAGE recipe → reconstructed font/image | `call glyph_compose --json '{"kage_json":…}'` | recipe → `gwz:sha256:…`, `reconstruction_from_bitmap` |
| Submit transcription parts | `transcribe FILE` | JSON parts → stored |
| Ordered reading result | `document result/export` | document → JSON/HTML |

After **any** call, read its result, then choose the next action. Never assume text, reading order, encoding, class names or boundaries are correct. `--detach`/`status … --wait --output` keep long jobs resumable; `call` escapes to any schema.

## Prepare evidence before reading (full-PDF entry point)

`document open FILE` uploads original bytes and returns `document_id` and page count (`upload FILE` alone returns `input_key` for later `--input-key`). Page numbers are PDF page ordinals, not printed page numbers.

```sh
guwenzi-tools page prepare DOCUMENT_ID 1 --output work/page-1 --detach
guwenzi-tools status JOB_ID --wait --output work/page-1
```

Each page directory has `brief.json`, complete `result.json`, `materials.json`, images and `index.html`. Read the brief, then view the actual page/layout/line images. Native text is source text with unverified encoding; raster text awaits your transcription. `evidence_prepared`, `resources_present` and completed jobs do not mean correct text.

With `--no-images`, JSON/metadata are saved and images can be fetched later:

```sh
guwenzi-tools asset 'asset:SHA256…' --output work/img.png          # original
guwenzi-tools asset 'asset:SHA256…' --view --output work/view.png  # enlarged view
```

Original bytes are hash-verified; nearest-neighbor enlargement adds no new stroke detail.

## Resolve uncertain forms

Use an original line/block `asset_id`. `line split` produces projection slot proposals; it does not prove one crop = one whole character. If a glyph is cut or neighbors merged, view the original line and `crop` a corrected box: half-open `[left, top, right, bottom]` in **original** pixels.

```sh
guwenzi-tools crop ASSET_ID --box 10 0 60 80 --output work/crop
guwenzi-tools glyph search CROP_ASSET_ID --output work/search
```

The search dir holds the query plus each candidate's original and enlarged image with temporary `cN` IDs. Compare actual strokes with the image tool. Similarity is not a probability; ranking alone is not identity; source labels stay hidden until selection.

```sh
guwenzi-tools glyph select SEARCH_ID c3 --output work/binding.json
```

Returns `binding_id`, permanent code and source identity `model_selected_unverified`. Candidate IDs are local to that search. No match → expand once: `glyph search CROP_ASSET_ID --expand-search SEARCH_ID --output work/expanded` (already-seen candidates are excluded). Still uncertain → `glyph register CROP_ASSET_ID` preserves the original under `gwz:sha256:…` identifying those bytes, not a decoded character.

`glyph resolve CODE --output dir` returns identity: stdout summary shows source codepoints, font hash, glyph ID/name, version, aliases/relations; full metadata in `result.json`. `reading_status=no_confirmed_reading_in_v1` is not a reading. Keep borrowed/unassigned/PUA codes with their exact source font identity; never turn a codepoint or glyph name into an asserted interpretation. `source_material_in_bundle=false` means identifying metadata exists but the original font program is not in this deployment.

## Transcribe, interpret and export

Fill the page's `transcriptions.draft.json`: each request has `document_id`, `page`, `block_id`, ordered `parts`; each part has exactly one of `text`, `binding_id`, `glyph_code` (registered unresolved), `asset_id`. Structured parts, never code markers typed into text. Preserve punctuation, non-BMP chars, PUA and variant sequences; no Unicode normalization, no traditional/simplified conversion.

```sh
guwenzi-tools transcribe FILE [--skip-empty]
guwenzi-tools document result DOCUMENT_ID --offset 0 --limit 20   # follow next_offset
```

Bindings must originate from the target block or its recorded crops/slots — the host validates scope. Transcribing to Unicode and identifying the precise font/source asset are different claims; bracketed author readings are evidence to interpret, not identity labels.

```sh
guwenzi-tools document export DOCUMENT_ID --output work/export [--no-images]
```

Export saves ordered JSON/JSONL, Markdown/HTML, the original document and API-referenced images/path fragments. Unreferenced server intermediates are not claimed as exported.

`reading.html` shows preserved glyphs inline instead of long markers. For supported original PDF paths (install `guwenzi-tools[fonts]` first): `font build --packet work/page-1 --output work/font` produces a real OpenType font plus `glyph-map.json`; `font encode text.txt --map work/font/glyph-map.json --output work/typeset.txt` maps `⟦gwz:…⟧` markers to font-local PUA. Keep font+SHA+mapping together. Parenthesized readings are not replacement glyphs.

If companion CLI `guwenzi-glyph` is separately installed, printed bitmaps may be reconstructed via its KAGE `compose` recipe with visual revise loops; label output “据图重建”, preserve the original and exact component versions, and never claim recovery of the source font. Rubbings and handwriting stay images. See that CLI's README.

When explaining a paper, separate author statements, observed shape details and your inference; cite page/block IDs; preserve unresolved glyphs and figure/table references; check reading order and suspected missing text against the full page. For anything not wrapped above, `call TOOL --json @request.json` with the schema from `tools`.
