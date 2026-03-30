# Deep-Learning_SP26_Midterm-Project

## SVG Generation from Text Prompts

## Author
**Tianqi Zhen**

---

## Project Overview

This project focuses on generating SVG (Scalable Vector Graphics) code from natural language prompts using a fine-tuned language model. The objective is to produce **valid, compact, and visually meaningful SVG outputs** that satisfy strict formatting and structural constraints.

The task is based on a Kaggle competition, where the model is trained on prompt–SVG pairs and evaluated on its ability to generate valid SVGs for unseen prompts.

---

## Task Description

Given a natural language prompt such as:

> "A black rectangle enclosed in a circle"

the model is expected to generate valid SVG code like:

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
* Must use only allowed SVG tags
* Must render meaningful visual shapes

---

## Model

* **Base model:** Qwen2.5
* **Fine-tuning method:** LoRA (Low-Rank Adaptation)
* **Training framework:** transformers, TRL (SFTTrainer), PEFT

---

## Data Processing

The project applies several preprocessing steps:

1. Remove invalid SVG samples
2. Filter out disallowed tags
3. Limit token length for efficient training
4. Construct a shorter dataset for faster experimentation

---

## SVG Post-processing Pipeline

The most critical component of this project is the **SVG post-processing pipeline**, which ensures that generated outputs are valid and usable.

---

### 1. SVG Extraction

`extract_first_svg()`

* Removes code blocks and special tokens
* Extracts the first `<svg>...</svg>` block
* Attempts to recover incomplete SVG outputs

---

### 2. SVG Repair

`repair_common_svg_issues()`

Fixes common generation errors, including:

* Broken `<svg>` tags
* Missing closing tags (`</path>`, `</g>`, etc.)
* Truncated tags (`</pa`, `</pat`)
* Empty `fill` attributes

---

### 3. Canvas Enforcement

`enforce_canvas()`

Ensures required attributes:

* `xmlns="http://www.w3.org/2000/svg"`
* `width="256"`
* `height="256"`
* `viewBox="0 0 256 256"`

---

### 4. Validation and Sanitization

`sanitize_generated_svg()`

Pipeline steps:

1. Extract SVG
2. Normalize formatting
3. Fix fill-related issues
4. Validate SVG structure
5. Repair and revalidate
6. Apply fallback if still invalid

---

### 5. Fallback Mechanism

`fallback_svg()`

If all validation attempts fail, return:

```xml
<svg ...>
  <rect width="256" height="256" fill="white"/>
</svg>
```

This guarantees:

* Valid submission format
* No missing rows
* No runtime failures during evaluation

---

### 6. Visible Shape Check

`has_visible_shape()`

Ensures the SVG is not empty by checking for visible elements such as:

* `<path>`
* `<rect>`
* `<circle>`
* `<polygon>`

---

## Inference Strategy

SVG generation is handled by:

`generate_best_svg()`

### Key ideas:

* Multiple attempts (`tries`)
* Selection of the best candidate based on:

  * Presence of visible shapes
  * Number of path elements
  * SVG length

### Scoring function:

```python
score = (shape_score, path_count, length)
```

---

## Results

* Successfully generated **1000 SVG outputs**
* All outputs are valid and properly formatted
* Established a working baseline on the Kaggle leaderboard

---

## How to Run

1. Open the notebook in **Google Colab**
2. Upload the dataset (`dl-spring-2026-svg-generation.zip`)
3. Install required dependencies
4. Run inference cells to generate `submission.csv`

---

## Future Work

* Improve generation of non-fallback SVGs
* Enhance geometric accuracy of shapes
* Optimize decoding strategy
* Explore larger models (e.g., 4B models)

---
