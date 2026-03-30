# Deep-Learning_SP26_Midterm-Project_SVG-Generation-from-Text-Prompts
## Project Overview
This project focuses on generating SVG  code from natural language prompts using a fine-tuned language model. The goal is to produce valid, compact, and visually meaningful SVG outputs that satisfy strict formatting and structural constraints. The task is based on a Kaggle competition where the model is trained on prompt-SVG pairs and evaluated on its ability to generate valid SVGs for unseen prompts.

## Task
Given a natural language prompt such as:
"A black rectangle enclosed in a circle"
the model must generate valid SVG code like:
<svg ...>
  <circle ... />
  <rectangle ... />
</svg>
Constraints include:
Must be valid XML
Must start with <svg> and end with </svg>
Must be under 8000 characters
Must use allowed SVG tags
Must render meaningful visual shapes

## Model
Base model: Qwen2.5
Fine-tuning method: LoRA
Training framework: transformers, trl (SFTTrainer), peft

## Data Processing
The project applied multiple preprocessing steps:
1. Removed invalid SVG samples
2. Filtered disallowed tags
3. Limited token length for efficient training
4. Built a short dataset for faster training

## Core Pipeline
The most important part of this project is the SVG post-processing pipeline

### SVG Extraction
extract_first_svg()
Removes code blocks and special tokens
Extracts the first <svg>...</svg> block
Attempts to recover incomplete SVG outputs

### SVG Repair
repair_common_svg_issues()
Fixes common generation errors include:
Broken <svg> tags
Missing closing tags (</path>, </g>, etc.)
Truncated tags (</pa, </pat)
Empty fill attributes

### Canvas Enforcement
enforce_canvas()
Ensures required attributes:
xmlns="http://www.w3.org/2000/svg"
width="256"
height="256"
viewBox="0 0 256 256"

### Validation and Sanitization
sanitize_generated_svg()
Pipeline:
Extract SVG
Normalize formatting
Fix fill issues
Validate SVG structure
Repair and revalidate
Fallback if still invalid

### Fallback
fallback_svg()
If all attempts fail, return:
<svg ...>
  <rect width="256" height="256" fill="white"/>
</svg>
This guarantees:
Valid submission
No missing rows
No crash during evaluation

### Visible Shape Check
has_visible_shape()
Ensures SVG is not empty by checking for path, rect, circle, etc.

### Inference Strategy
SVG generation uses:
generate_best_svg()
The factors are include:
Multiple attempts (tries)
Select best candidate based on:
Presence of shapes
Number of paths
SVG length

Scoring:
score = (shape_score, path_count, length)

### Results
1. Successfully generated 1000 SVG outputs
2. All rows valid and formatted correctly
3. Established a working baseline on Kaggle

## HOW TO RUN
1. Open the notebook in Google Colab
2. upload the 'dl-spring-2026-svg-generation' zip file
3. Install dependencies
4. Run inference to generate submission.csv
