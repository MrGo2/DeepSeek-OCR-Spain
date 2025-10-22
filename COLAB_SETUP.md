# 🚀 DeepSeek-OCR Colab Setup Guide

**NEW!** PDF support added - process entire documents page-by-page! 📄

## Quick Start (5 minutes)

### 1. Upload to Colab
1. Go to [colab.research.google.com](https://colab.research.google.com)
2. **File → Upload notebook**
3. Select `DeepSeek_OCR_Colab_with_PDF.ipynb` from this folder (or use `DeepSeek_OCR_Colab.ipynb` for images only)
4. Wait for upload to complete

### 2. Enable GPU
1. **Runtime → Change runtime type**
2. Select **T4 GPU** (or V100/A100 if available)
3. Click **Save**

### 3. Run Everything
1. **Runtime → Run all**
2. First run: ~15-20 min (downloads model)
3. Cached runs: ~2-3 min

---

## What the Notebook Does

### ✅ Automatic Setup
- Checks GPU availability
- Installs PyTorch 2.6.0 + CUDA 11.8
- Installs Flash Attention 2.7.3
- Downloads DeepSeek-OCR model (~8GB, cached after first run)

### ✅ Sample Tests
Tests 3 different modes:
1. **Markdown conversion** (with layout)
2. **Free OCR** (text only)
3. **Image description** (general understanding)

### ✅ Your Own Images
- Upload via file picker
- Batch processing
- Download all results as ZIP

### ✅ PDF Processing (NEW!)
- Upload PDF documents
- Automatic page-by-page conversion
- Batch process all pages
- Compile into single markdown file
- Page separators and headers
- Quality control (DPI adjustable)

---

## Expected Timeline

| Step | First Run | Cached Run |
|------|-----------|------------|
| GPU check | 5 sec | 5 sec |
| Clone repo | 10 sec | 10 sec |
| Install PyTorch | 2 min | 30 sec |
| Install deps | 2 min | 30 sec |
| Install Flash Attn | 3-5 min | 30 sec |
| Download model | 5-10 min | 30 sec |
| Run tests | 2-3 min | 2-3 min |
| PDF (10 pages) | 2-3 min | 2-3 min |
| **TOTAL** | **~15-20 min** | **~5 min** |

**PDF Processing Time**:
- 1 page: ~10-15 seconds
- 10 pages: ~2-3 minutes
- 50 pages: ~10-15 minutes
- 100 pages: ~20-30 minutes

---

## Customization

### Change Resolution Mode

Edit the `model.infer()` parameters:

```python
# Fastest (Tiny - 64 tokens)
base_size=512, image_size=512, crop_mode=False

# Fast (Small - 100 tokens)
base_size=640, image_size=640, crop_mode=False

# Balanced (Base - 256 tokens)
base_size=1024, image_size=1024, crop_mode=False

# Best Quality (Large - 400 tokens)
base_size=1280, image_size=1280, crop_mode=False

# Best for Documents (Gundam - dynamic)
base_size=1024, image_size=640, crop_mode=True  # ← DEFAULT
```

### Change Prompt

```python
# Documents to markdown
PROMPT = "<image>\n<|grounding|>Convert the document to markdown."

# General OCR
PROMPT = "<image>\n<|grounding|>OCR this image."

# Text only (no layout)
PROMPT = "<image>\nFree OCR."

# Parse figures
PROMPT = "<image>\nParse the figure."

# Describe image
PROMPT = "<image>\nDescribe this image in detail."

# Locate specific text
PROMPT = "<image>\nLocate <|ref|>your text here<|/ref|> in the image."
```

### PDF Quality Settings

Adjust DPI in the PDF processing cell:

```python
page_images = pdf_to_images(pdf_bytes, dpi=144)  # ← Change this

# dpi=72:  Fast, lower quality (good for text-only docs)
# dpi=144: Balanced (default - recommended)
# dpi=300: Slow, high quality (for scanned documents)
```

**Trade-offs**:
- Higher DPI = Better OCR accuracy but slower + more GPU memory
- Lower DPI = Faster but may miss small text

---

## Troubleshooting

### ❌ "No GPU found"
**Solution**: Runtime → Change runtime type → Select GPU → Save

### ❌ "Out of memory"
**Solution**:
1. Runtime → Disconnect and delete runtime
2. Restart with smaller `base_size`:
   ```python
   base_size=640, image_size=640, crop_mode=False
   ```

### ❌ "Model download stuck"
**Solution**:
1. Restart runtime
2. Check HuggingFace status: [status.huggingface.co](https://status.huggingface.co)

### ❌ "Flash Attention build failed"
**Solution**: Already fixed in notebook - uses `--no-build-isolation`

---

## Cost Breakdown

| Plan | Monthly Cost | GPU Access | Max Runtime |
|------|--------------|------------|-------------|
| **Free** | $0 | Limited (K80/T4) | 12h |
| **Colab Pro** | $10 | Priority (T4/V100) | 24h |
| **Colab Pro+** | $50 | Highest (V100/A100) | 24h |

**You have Colab Pro** → No extra cost! 🎉

---

## Output Format

Results saved in `user_outputs/`:
```
user_outputs/
├── your_image.jpg_result.md
├── another_image.png_result.md
└── result.mmd
```

Download as ZIP from last cell.

---

## Next Steps

### Option 1: Keep Using Colab
- Works great for testing
- $0 additional cost (you have Pro)
- 24h runtime limit

### Option 2: Move to Azure (Production)
- For 24/7 API
- Enterprise integration
- Auto-scaling

### Option 3: Deploy to Railway/Fly.io
- Simple GPU deployment
- API endpoint
- ~$0.50/hr

---

## Support

**Issues?** Share:
1. Which cell failed
2. Error message
3. Screenshot

**Questions?** Check:
- [DeepSeek-OCR README](https://github.com/deepseek-ai/DeepSeek-OCR)
- [Paper](https://arxiv.org/abs/2510.18234)

---

**Created**: October 2025
**Author**: Carlos Lorenzo Santos
**Model**: [deepseek-ai/DeepSeek-OCR](https://huggingface.co/deepseek-ai/DeepSeek-OCR)
