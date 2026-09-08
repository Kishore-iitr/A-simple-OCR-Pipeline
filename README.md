# OCR Pipeline

Runs the same test images through multiple OCR approaches and compares their
outputs using a common word-level similarity metric.

**Open `OCR_pipeline_simplr.ipynb`** — the notebook contains the complete
pipeline, preprocessing steps, OCR inference, evaluation and comparison.

## What's compared

| # | Method                                                 | Type                        |
| - | ------------------------------------------------------ | --------------------------- |
| 1 | **Tesseract** (`pytesseract`)                          | Classical OCR + LSTM        |
| 2 | **EasyOCR**                                            | Deep learning: CRAFT + CRNN |
| 3 | **Vision-Language Model** (Qwen 2.5-VL via OpenRouter) | Multimodal LLM              |

## Test images

Three images generated from known source text:

* `noisy_document.png` — printed invoice with blur, noise, low contrast and skew
* `handwriting_cursive.png` — regular cursive handwriting
* `handwriting_dkg.png` — irregular handwriting

## Preprocessing Strategy

For Tesseract, the image passes through:

```text
Input → Grayscale → Upscaling → Denoising → Deskewing
      → Otsu Thresholding → OCR
```

The preprocessing is designed to improve recognition under blur, noise,
rotation and low-contrast conditions.

EasyOCR performs its own text detection and recognition, while the VLM receives
the image directly and is prompted to transcribe the visible text.

## Evaluation

Since the source text is known, OCR output is compared against the ground truth
using `difflib.SequenceMatcher` on **word sequences**.

* `1.0` → identical word sequence
* `0.0` → no similarity

This avoids penalizing minor formatting or spacing differences as heavily as
character-level exact matching.

## Results

| Image                      | Tesseract | EasyOCR | VLM  |
| -------------------------- | --------: | ------: |----: |
| Printed invoice (degraded) | **78.8%** |   51.4% | 100% |
| Neat handwriting           | **86.7%** |   53.3% | 100% |
| Messy handwriting          | **26.7%** |   14.8% | 100% |

The VLM section is included in the notebook but requires an
`OPENROUTER_API_KEY` and internet access to run.

## Bottom Line

* **Tesseract** — lightweight, offline and effective after preprocessing.
* **EasyOCR** — end-to-end detection + recognition with less manual preprocessing.
* **VLM** — flexible and potentially more robust to difficult real-world images,
  but requires API access and has higher deployment cost.

## Tech Stack

**Python · OpenCV · Tesseract · EasyOCR · PyTorch · Qwen 2.5-VL · OpenRouter**
