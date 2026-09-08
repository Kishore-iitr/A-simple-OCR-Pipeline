# OCR Framework Shootout


## What's compared

| # | Method | Type |
|---|--------|------|
| 1 | **Tesseract** (`pytesseract`) | Classical pipeline + LSTM recognizer |
| 2 | **EasyOCR** | Deep learning: CRAFT detector + CRNN recognizer |
| 3 | **TrOCR** (Microsoft) | Pure Transformer (ViT encoder + text decoder), separate checkpoints for printed vs. handwritten text | (not included currently)
| 4 | **A vision-language model**, via OpenRouter | General-purpose multimodal LLM, not a dedicated OCR model at all |

## Test images

Three images rendered from known source text (so exact ground truth is
available to score against):

- `sample_images/noisy_document.png` — a printed invoice, rotated, blurred, low-contrast, noisy
- `sample_images/handwriting_cursive.png` — a to-do list in a regular, evenly-formed cursive font
- `sample_images/handwriting_dkg.png` — the same list in a messier, irregular handwriting-style font

## Accuracy strategy

Character-level exact match is too harsh for OCR (it penalizes spacing
differences as if they were wrong letters). Every score in this notebook is a
**word-level similarity ratio** (`difflib.SequenceMatcher` on the word
sequences, 1.0 = identical).

Two reference points are used, because most real projects only have one of them:

1. **vs. known ground truth** — the trustworthy score, used here because we
   generated the images ourselves and know the exact correct text.
2. **vs. the VLM's output** — the practical stand-in for ground truth when
   you're processing real-world images with no manual labels: treat a
   strong general-purpose model's transcription as a pseudo-reference, and
   measure how closely a cheaper/faster/local tool tracks it.

## Results 

|Framework|handwriting\_cursive\.png|handwriting\_dkg\.png|noisy\_document\.png|
|---|---|---|---|
|EasyOCR|53\.3%|14\.8%|46\.6%|
|Tesseract|53\.3%|53\.3%|93\.9%|
|VLM |100\.0%|100\.0%|100\.0%|

## Bottom line

- **Tesseract** — free, tiny, no GPU. Fine for clean printed text once
  preprocessed properly; weak on handwriting.
- **EasyOCR** — less manual preprocessing needed (built-in detection *and*
  recognition), good general starting point; also not a handwriting specialist.
- **TrOCR** — the only local model here with checkpoints actually trained on
  handwriting. Recognition-only, so it needs a separate line-detection step
  first (built in this notebook as `segment_lines()` — a simple heuristic,
  not a learned detector, which is exactly why production frameworks ship
  one instead).
- **VLM (OpenRouter)** — no preprocessing, no line segmentation, no
  printed-vs-handwritten model choice; one API call handles all of it and
  tends to be the most robust to real-world image quality issues. Costs
  money per call and needs internet.



