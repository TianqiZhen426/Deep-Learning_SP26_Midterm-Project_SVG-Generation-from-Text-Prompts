# Deep-Learning_SP26_Midterm-Project

## Author
**Tianqi Zhen**

## SVG Generation from Text Prompts

---

## Project Overview

This project aims to generate SVG code from natural language prompts using a fine-tuned language model. The goal is to produce valid, compact, and visually meaningful SVG outputs under strict structural constraints. The task is based on a Kaggle competition where models are trained on prompt–SVG pairs and evaluated on their ability to generate valid SVGs for unseen prompts. The key challenge of this project is that language models often generate invalid SVGs, including malformed XML, missing tags, or additional explanatory text. Therefore, a major contribution of this work is a robust SVG post-processing and repair pipeline.

---

## Task Description

Given a natural language prompt such as:

> "A black rectangle enclosed in a circle"

the model must generate valid SVG code:

```xml
<svg ...>
  <circle ... />
  <rect ... />
</svg>
```

### Constraints

* Must be valid XML
* Must start with `<svg>` and end with `</svg>`
* Must be under 8000 characters
* Must use allowed SVG tags
* Must contain meaningful visual content

---

## Model

* **Base model:** Qwen2.5 (small variant)
* **Fine-tuning method:** LoRA (Low-Rank Adaptation)
* **Frameworks:** transformers, TRL (SFTTrainer), PEFT

The model is fine-tuned to map natural language prompts to SVG code.

---

## Data Processing

To improve training stability and efficiency, we applied:

1. Removal of invalid SVG samples
2. Filtering of disallowed tags
3. Token length constraints to avoid overly long sequences
4. Construction of a short dataset (≤1536 tokens)

This reduced training cost significantly while maintaining reasonable performance.

---

## SVG Post-processing Pipeline (Core Contribution)

A major challenge is that raw model outputs are often not directly usable.
To address this, a multi-stage pipeline applied:

---

### 1. SVG Extraction

`extract_first_svg()`

* Removes code fences and special tokens
* Extracts the first `<svg>...</svg>` block
* Attempts to recover incomplete outputs

---

### 2. SVG Repair

`repair_common_svg_issues()`

Fixes common errors:

* Broken `<svg>` tags
* Missing closing tags (`</path>`, `</g>`)
* Truncated tags (`</pa`, `</pat`)
* Empty fill attributes

---

### 3. Canvas Enforcement

`enforce_canvas()`

Ensures all outputs satisfy:

* `xmlns="http://www.w3.org/2000/svg"`
* `width="256"`
* `height="256"`
* `viewBox="0 0 256 256"`

---

### 4. Validation and Sanitization

`sanitize_generated_svg()`

Pipeline:

1. Extract SVG
2. Normalize formatting
3. Fix fill issues
4. Validate structure
5. Repair and revalidate
6. Fallback if still invalid

---

### 5. Visible Shape Check

`has_visible_shape()`

Ensures the SVG is not empty by checking for:

* `<path>`
* `<rect>`
* `<circle>`
* `<polygon>`

---

### 6. Fallback Mechanism

`fallback_svg()`

If all attempts fail:

```xml
<svg ...>
  <rect width="256" height="256" fill="white"/>
</svg>
```

This guarantees:

* Valid submission
* No missing rows
* No runtime errors

---

## Inference Strategy

SVG generation is handled by:

`generate_best_svg()`

### Strategy

* Generate SVG using the model
* Apply full sanitization pipeline
* Score candidates based on:

```python
score = (shape_score, path_count, length)
```

Where:

* `shape_score`: whether visible shapes exist
* `path_count`: number of graphical elements
* `length`: SVG richness

---

## Engineering Considerations

### Runtime Constraints

Generating 1000 SVGs in Google Colab introduces significant constraints:

* Long inference time (around 12 hours)
* Frequent runtime disconnections
* GPU session limits

---

### Speed vs Quality Trade-off

To ensure successful submission within limited resources and time, I have to do the following trade-offs:

#### Optimizations for Speed

* Reduced `max_new_tokens` from ~1800 → **256** (Original: 1800)
* Set `tries = 1` (Original: 3)
* Used a smaller training subset
* Performed incremental saving during inference

#### Impact on Quality

These choices led to:

* Increased fallback SVG outputs
* Reduced complexity of generated shapes
* Lower diversity in SVG structures
* Fewer multi-element compositions

The model became **more stable but less expressive**.

---

## Results

* Successfully generated 1000 valid SVG outputs
* All rows satisfied format constraints
* Achieved a working baseline on Kaggle
* A noticeable portion of outputs rely on fallback
* Visual complexity is limited

---

## Limitations

1. High reliance on fallback SVGs
2. Limited structural diversity
3. Model occasionally generates non-SVG text
4. Performance constrained by Colab runtime

---

## Future Improvements

To improve performance, the following directions are proposed:

### 1. Improve Generation Quality

* Increase `max_new_tokens` size
* Use multiple sampling attempts (`tries = 3`)
* Apply better decoding strategies

### 2. Stronger Models

* Upgrade to larger models (4B)
* Use instruction-tuned variants

### 3. Better Training Strategy

* Use larger filtered datasets
* Apply curriculum learning (short → long SVGs)

### 4. Reduce Fallback Dependency

* Improve repair logic
* Add structure-aware constraints during generation

### 5. Efficiency Improvements

* Batch inference
* More stable execution environment (outside Colab)

---

## How to Run

1. Open the notebook in **Google Colab**
2. Upload dataset (`dl-spring-2026-svg-generation.zip`)
3. Install dependencies
4. Run all the code blocks
5. Run inference to generate `submission.csv`

---
