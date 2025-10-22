# DeepSeek-OCR Complete Options & Features Guide

Comprehensive guide to all DeepSeek-OCR parameters, modes, and use cases.

---

## Table of Contents

1. [Resolution Modes](#resolution-modes)
2. [Crop Modes](#crop-modes)
3. [PDF Quality Settings](#pdf-quality-settings)
4. [Prompt Types](#prompt-types)
5. [Use Case Recommendations](#use-case-recommendations)
6. [Performance Tuning](#performance-tuning)
7. [Troubleshooting Guide](#troubleshooting-guide)

---

## Resolution Modes

DeepSeek-OCR supports 5 resolution modes that balance speed, quality, and GPU memory usage.

### Parameters

```python
model.infer(
    tokenizer,
    prompt=prompt,
    image_file=image_path,
    base_size=1024,      # ← Controls base resolution
    image_size=640,      # ← Controls crop tile size
    crop_mode=True,      # ← Enable/disable dynamic cropping
    ...
)
```

---

### Mode 1: Tiny (Fastest)

```python
base_size=512
image_size=512
crop_mode=False
```

**Characteristics**:
- **Vision tokens**: 64
- **Speed**: ~5 seconds per image
- **GPU memory**: ~8 GB
- **Quality**: Lowest

**When to use**:
- ✅ Quick previews
- ✅ Low-resolution documents
- ✅ Text-only simple layouts
- ✅ When speed is critical
- ✅ Limited GPU memory (<12 GB)

**When NOT to use**:
- ❌ High-resolution scans
- ❌ Complex layouts
- ❌ Small fonts (<10pt)
- ❌ Tables with dense data

**Example use cases**:
- Receipt scanning
- Simple forms
- Quick text extraction
- Batch processing thousands of images

---

### Mode 2: Small (Fast)

```python
base_size=640
image_size=640
crop_mode=False
```

**Characteristics**:
- **Vision tokens**: 100
- **Speed**: ~7 seconds per image
- **GPU memory**: ~10 GB
- **Quality**: Good

**When to use**:
- ✅ Standard documents
- ✅ Single-column text
- ✅ Basic tables
- ✅ Moderate detail requirements
- ✅ Balanced speed/quality

**When NOT to use**:
- ❌ Multi-column layouts
- ❌ Very small text
- ❌ Complex diagrams
- ❌ High-accuracy requirements

**Example use cases**:
- Letters
- Simple reports
- Email screenshots
- Social media posts

---

### Mode 3: Base (Balanced)

```python
base_size=1024
image_size=1024
crop_mode=False
```

**Characteristics**:
- **Vision tokens**: 256
- **Speed**: ~10 seconds per image
- **GPU memory**: ~12 GB
- **Quality**: High

**When to use**:
- ✅ Standard documents
- ✅ Moderate complexity
- ✅ Good balance needed
- ✅ Most general use cases
- ✅ When Gundam is overkill

**When NOT to use**:
- ❌ Very large images (>2000px)
- ❌ Multi-page layouts
- ❌ Extreme detail needed

**Example use cases**:
- Business documents
- Academic papers (single column)
- Slides/presentations
- Standard PDFs

---

### Mode 4: Large (High Quality)

```python
base_size=1280
image_size=1280
crop_mode=False
```

**Characteristics**:
- **Vision tokens**: 400
- **Speed**: ~15 seconds per image
- **GPU memory**: ~14 GB
- **Quality**: Very high

**When to use**:
- ✅ High-resolution documents
- ✅ Fine details critical
- ✅ Technical drawings
- ✅ Dense information
- ✅ Accuracy over speed

**When NOT to use**:
- ❌ Batch processing
- ❌ Time-sensitive tasks
- ❌ Limited GPU memory
- ❌ Simple documents

**Example use cases**:
- Technical manuals
- Medical records
- Legal documents
- Blueprints
- High-res scans

---

### Mode 5: Gundam (Dynamic/Adaptive) ⭐ **RECOMMENDED**

```python
base_size=1024
image_size=640
crop_mode=True
```

**Characteristics**:
- **Vision tokens**: 256 + n×100 (dynamic)
- **Speed**: ~10-15 seconds per image
- **GPU memory**: ~12-16 GB (varies)
- **Quality**: Excellent for documents

**How it works**:
1. Processes full image at 1024×1024 (base)
2. Splits into 640×640 tiles for detail
3. Adaptively adjusts based on aspect ratio
4. Combines global + local features

**When to use**: ✅ **DEFAULT FOR DOCUMENTS**
- ✅ Multi-column layouts
- ✅ Complex documents
- ✅ Academic papers
- ✅ Magazines
- ✅ Infographics
- ✅ Mixed content (text + images)
- ✅ Any document layout

**When NOT to use**:
- ❌ Very simple single-column text (use Small/Base instead)
- ❌ Limited GPU memory
- ❌ Need fastest speed

**Example use cases**:
- Research papers
- Textbooks
- Newspapers
- Technical reports
- Scientific journals
- Any PDF document

**Dynamic behavior**:
```
640×480 image    → 1 base view only (256 tokens)
1280×1024 image  → 1 base + 2×2 tiles (256 + 400 = 656 tokens)
2560×1920 image  → 1 base + 4×3 tiles (256 + 1200 = 1456 tokens)
```

---

## Crop Modes

### crop_mode=False (Fixed Resolution)

**Behavior**:
- Resizes entire image to `base_size × base_size`
- Single pass through model
- No tiling

**Pros**:
- ✅ Faster processing
- ✅ Lower memory usage
- ✅ Predictable token count
- ✅ Good for simple layouts

**Cons**:
- ❌ May lose detail in large images
- ❌ Poor for complex layouts
- ❌ Aspect ratio distortion

**Use when**:
- Image resolution ≤ base_size
- Simple single-column layouts
- Speed is priority
- GPU memory is limited

---

### crop_mode=True (Dynamic Tiling) ⭐ **RECOMMENDED**

**Behavior**:
- Processes base view at `base_size`
- Splits into `image_size × image_size` tiles
- Intelligently crops based on aspect ratio
- Preserves detail

**Pros**:
- ✅ Best quality for complex layouts
- ✅ Preserves fine details
- ✅ Handles large images
- ✅ No aspect ratio distortion
- ✅ Adaptive to content

**Cons**:
- ❌ Slower (more tiles)
- ❌ Higher memory usage
- ❌ Variable token count

**Use when**:
- **Default for documents**
- Image resolution > 1024px
- Multi-column layouts
- Complex document structure
- Accuracy is priority

---

## PDF Quality Settings

### DPI (Dots Per Inch)

Controls resolution when converting PDF pages to images.

```python
page_images = pdf_to_images(pdf_bytes, dpi=144)
```

---

### DPI=72 (Fast/Draft)

**Characteristics**:
- **Speed**: Fastest (~5 sec/page)
- **Memory**: Low (~8 GB)
- **Quality**: Basic
- **File size**: Small

**When to use**:
- ✅ Text-only PDFs
- ✅ Large PDFs (100+ pages)
- ✅ Draft/preview mode
- ✅ Simple layouts
- ✅ Memory constraints

**When NOT to use**:
- ❌ Scanned documents
- ❌ Small fonts
- ❌ Detailed diagrams
- ❌ High accuracy needed

**Example documents**:
- Plain text PDFs
- Simple reports
- Bulk processing
- Quick previews

---

### DPI=144 (Balanced) ⭐ **RECOMMENDED**

**Characteristics**:
- **Speed**: Moderate (~10 sec/page)
- **Memory**: Medium (~12 GB)
- **Quality**: Good
- **File size**: Medium

**When to use**: ✅ **DEFAULT**
- ✅ Most PDFs
- ✅ Standard documents
- ✅ Born-digital PDFs
- ✅ Mixed content
- ✅ General use

**When NOT to use**:
- ❌ Very poor scans (use 300)
- ❌ Need extreme speed (use 72)

**Example documents**:
- Business documents
- Reports
- Presentations
- Standard PDFs
- eBooks

---

### DPI=300 (High Quality)

**Characteristics**:
- **Speed**: Slow (~20 sec/page)
- **Memory**: High (~16 GB)
- **Quality**: Excellent
- **File size**: Large

**When to use**:
- ✅ Scanned documents
- ✅ Poor quality originals
- ✅ Small text (<8pt)
- ✅ Detailed images
- ✅ Archival quality needed
- ✅ Legal/medical documents

**When NOT to use**:
- ❌ Clean digital PDFs
- ❌ Large PDFs (slow)
- ❌ Limited GPU memory
- ❌ Time-sensitive tasks

**Example documents**:
- Scanned books
- Faxes
- Old documents
- Microfilm scans
- Handwriting
- Technical drawings

---

## Prompt Types

Different prompts optimize for different extraction tasks.

### 1. Document to Markdown (Layout Preservation)

```python
prompt = "<image>\n<|grounding|>Convert the document to markdown."
```

**What it does**:
- Preserves document structure
- Converts headings to markdown (`#`, `##`)
- Maintains lists, tables
- Keeps formatting
- Adds bounding boxes for images

**Best for**:
- ✅ Structured documents
- ✅ Academic papers
- ✅ Reports with sections
- ✅ Technical documentation
- ✅ Books/chapters
- ✅ PDFs with layouts

**Output example**:
```markdown
# Main Title

## Section 1

This is body text with **bold** and *italic*.

- List item 1
- List item 2

| Column 1 | Column 2 |
|----------|----------|
| Data 1   | Data 2   |
```

---

### 2. General OCR (Grounded)

```python
prompt = "<image>\n<|grounding|>OCR this image."
```

**What it does**:
- Extracts all text
- Maintains spatial relationships
- Some layout awareness
- Bounding boxes included

**Best for**:
- ✅ Screenshots
- ✅ Mixed content images
- ✅ Infographics
- ✅ Slides
- ✅ Web pages
- ✅ Social media posts

**Output example**:
```
Header Text
Main content goes here
Footer information
```

---

### 3. Free OCR (Text Only)

```python
prompt = "<image>\nFree OCR."
```

**What it does**:
- Extracts pure text
- **No layout preservation**
- No formatting
- No bounding boxes
- Reading order only

**Best for**:
- ✅ When layout doesn't matter
- ✅ Plain text extraction
- ✅ Search indexing
- ✅ Text analysis
- ✅ Simple documents
- ✅ Fastest processing

**Output example**:
```
All the text from the document in reading order without any formatting or structure
```

---

### 4. Parse Figure

```python
prompt = "<image>\nParse the figure."
```

**What it does**:
- Focuses on diagrams/charts
- Extracts visual data
- Describes figure elements
- May extract chart data

**Best for**:
- ✅ Charts/graphs
- ✅ Diagrams
- ✅ Flowcharts
- ✅ Scientific figures
- ✅ Infographics
- ✅ Visual data extraction

**Output example**:
```
[Description of chart type]
X-axis: [labels]
Y-axis: [labels]
Data points: [values]
```

---

### 5. Detailed Description

```python
prompt = "<image>\nDescribe this image in detail."
```

**What it does**:
- General image understanding
- Describes content, layout, style
- Not focused on text extraction
- Visual analysis

**Best for**:
- ✅ Image analysis
- ✅ Scene understanding
- ✅ Content moderation
- ✅ Visual descriptions
- ✅ Accessibility (alt text)
- ✅ Not primarily OCR

**Output example**:
```
This is a business document with a header containing company logo and address. The main body includes three paragraphs of text, a table with financial data, and a signature at the bottom.
```

---

### 6. Locate Text (Grounding)

```python
prompt = "<image>\nLocate <|ref|>specific text<|/ref|> in the image."
```

**What it does**:
- Finds specific text
- Returns bounding boxes
- Spatial localization
- Multiple occurrences

**Best for**:
- ✅ Finding keywords
- ✅ Form field extraction
- ✅ Named entity location
- ✅ Redaction
- ✅ Highlighting
- ✅ Spatial search

**Output example**:
```
<|ref|>specific text<|/ref|><|det|>[[x1,y1,x2,y2]]<|/det|>
```

---

## Use Case Recommendations

### Use Case Matrix

| Document Type | Mode | Crop | DPI | Prompt |
|--------------|------|------|-----|--------|
| **Simple text PDF** | Small | False | 72 | Markdown |
| **Business document** | Base | True | 144 | Markdown |
| **Academic paper** | Gundam | True | 144 | Markdown |
| **Scanned book** | Large | True | 300 | Markdown |
| **Receipt** | Tiny | False | 144 | Free OCR |
| **Screenshot** | Small | False | - | General OCR |
| **Technical manual** | Gundam | True | 144 | Markdown |
| **Chart/graph** | Base | False | 144 | Parse figure |
| **Infographic** | Gundam | True | 144 | General OCR |
| **Legal document** | Large | True | 300 | Markdown |
| **Newspaper** | Gundam | True | 144 | Markdown |
| **Magazine** | Gundam | True | 144 | Markdown |
| **Handwritten notes** | Large | True | 300 | General OCR |
| **Form extraction** | Base | True | 144 | Locate text |
| **Social media post** | Small | False | - | General OCR |

---

## Performance Tuning

### Speed Priority

**Goal**: Process as fast as possible

```python
# Configuration
base_size=512
image_size=512
crop_mode=False
dpi=72  # For PDFs
prompt="<image>\nFree OCR."

# Expected performance
- Images: ~5 sec each
- PDFs: ~7 sec per page
- GPU memory: ~8 GB
```

**Trade-offs**:
- ⚠️ Lower accuracy
- ⚠️ May miss small text
- ⚠️ Poor for complex layouts

**Best for**:
- Bulk processing
- Simple documents
- Time-critical applications

---

### Quality Priority

**Goal**: Best possible accuracy

```python
# Configuration
base_size=1280
image_size=1280
crop_mode=True  # Use Gundam for documents
dpi=300  # For PDFs
prompt="<image>\n<|grounding|>Convert the document to markdown."

# Expected performance
- Images: ~20 sec each
- PDFs: ~25 sec per page
- GPU memory: ~16 GB
```

**Trade-offs**:
- ⚠️ Slower processing
- ⚠️ Higher memory usage
- ⚠️ More expensive compute

**Best for**:
- Critical documents
- Legal/medical
- Archival
- Research

---

### Balanced (Recommended)

**Goal**: Good quality, reasonable speed

```python
# Configuration
base_size=1024
image_size=640
crop_mode=True  # Gundam mode
dpi=144  # For PDFs
prompt="<image>\n<|grounding|>Convert the document to markdown."

# Expected performance
- Images: ~10 sec each
- PDFs: ~12 sec per page
- GPU memory: ~12 GB
```

**Trade-offs**:
- ✅ Good accuracy
- ✅ Reasonable speed
- ✅ Works for most use cases

**Best for**:
- ✅ **DEFAULT RECOMMENDATION**
- General documents
- Production workloads
- Most PDFs

---

## Troubleshooting Guide

### Issue: Out of Memory (OOM)

**Symptoms**:
```
CUDA out of memory
RuntimeError: CUDA OOM
```

**Solutions**:

1. **Reduce resolution**:
```python
base_size=640  # Instead of 1024
image_size=640
```

2. **Disable cropping**:
```python
crop_mode=False
```

3. **Lower PDF DPI**:
```python
dpi=72  # Instead of 144
```

4. **Process in batches**:
```python
# Process PDFs in chunks of 10 pages
for chunk in chunks(pages, 10):
    process(chunk)
```

5. **Restart runtime**:
- Runtime → Disconnect and delete runtime
- Runtime → Reconnect

---

### Issue: Slow Processing

**Symptoms**:
- Takes >30 seconds per image
- GPU utilization low

**Solutions**:

1. **Use smaller resolution**:
```python
base_size=640
crop_mode=False
```

2. **Reduce PDF quality**:
```python
dpi=72
```

3. **Check GPU**:
```python
!nvidia-smi  # Should show GPU usage
```

4. **Use simpler prompt**:
```python
prompt="<image>\nFree OCR."  # Fastest
```

---

### Issue: Poor OCR Accuracy

**Symptoms**:
- Missing text
- Incorrect characters
- Lost formatting

**Solutions**:

1. **Increase resolution**:
```python
base_size=1280
image_size=1280
```

2. **Enable cropping** (for documents):
```python
crop_mode=True
```

3. **Increase PDF DPI**:
```python
dpi=300
```

4. **Use appropriate prompt**:
```python
# For documents
prompt="<image>\n<|grounding|>Convert the document to markdown."
```

5. **Check image quality**:
- Ensure image is clear
- No blur/distortion
- Sufficient resolution

---

### Issue: Large PDF Processing

**Symptoms**:
- 100+ page PDFs
- Taking too long
- Memory issues

**Recommended settings**:

```python
# Fast processing
dpi=72
base_size=640
crop_mode=False

# Process in chunks
chunk_size = 10
for i in range(0, len(pages), chunk_size):
    chunk = pages[i:i+chunk_size]
    process_chunk(chunk)
```

**Alternative approach**:
```python
# Quality + Speed balance
dpi=144
base_size=1024
crop_mode=True
# But process pages 1-50, 51-100 separately
```

---

### Issue: Text Layout Lost

**Symptoms**:
- Text in wrong order
- Columns mixed up
- Tables broken

**Solutions**:

1. **Use Gundam mode**:
```python
base_size=1024
image_size=640
crop_mode=True
```

2. **Use layout-aware prompt**:
```python
prompt="<image>\n<|grounding|>Convert the document to markdown."
```

3. **Increase resolution**:
```python
base_size=1280  # For complex layouts
```

---

## Quick Reference

### Speed vs Quality Spectrum

```
FASTEST ←─────────────────────────────────→ BEST QUALITY

Tiny        Small       Base      Gundam      Large
512px       640px      1024px     1024+640   1280px
crop=F      crop=F     crop=F     crop=T     crop=T
dpi=72      dpi=72     dpi=144    dpi=144    dpi=300
5 sec       7 sec      10 sec     15 sec     20 sec
```

### Memory Requirements

```
Low Memory (8-10 GB)  → Tiny, Small
Medium (12-14 GB)     → Base, Gundam
High (16+ GB)         → Large, Gundam+300dpi
```

### Processing Time Guidelines

**Single Image**:
- Tiny: 5 sec
- Small: 7 sec
- Base: 10 sec
- Gundam: 15 sec
- Large: 20 sec

**PDF (per page)**:
- DPI=72: +2 sec
- DPI=144: +5 sec
- DPI=300: +10 sec

**Example**: 100-page PDF with Gundam + 144 DPI:
- Per page: 15 + 5 = 20 seconds
- Total: 100 × 20 = **~33 minutes**

---

## Summary: Best Practices

### ✅ Always Do

1. Start with **Gundam mode** (base=1024, image=640, crop=True)
2. Use **DPI=144** for PDFs
3. Use **markdown prompt** for documents
4. Test with 1-2 pages first
5. Monitor GPU memory usage

### ⚠️ Consider

1. Reduce settings if OOM errors
2. Increase DPI for scanned documents
3. Use crop=False for simple text-only docs
4. Process large PDFs in chunks

### ❌ Avoid

1. Using Tiny/Small for complex documents
2. Using DPI=300 for clean digital PDFs
3. Using crop=True with simple text (overkill)
4. Processing 100+ pages without chunking

---

**Document Version**: 1.0
**Last Updated**: October 2025
**Model**: DeepSeek-OCR
**Author**: Carlos Lorenzo Santos
