# Stroop-Style CLIP and VLM Testing

This project investigates whether vision-language models follow the **actual visual font color** in Stroop-style images or whether they become biased toward the **written word**.

The experiment is inspired by the human Stroop effect, where the meaning of a written color word can interfere with naming the ink color. In this project, the correct label is always the **font color**, but the image may contain conflicting cues such as the written word and background color.

---

## Project Overview

This repository contains two stages of experiments.

### Stage 1: Original CLIP White-Box Testing

The first stage performs both **black-box** and **white-box** testing on CLIP.

It focuses on the question:

> When the written word and the visual font color conflict, does CLIP predict the actual font color or follow the written word?

Example:

```text
Word shown: ORANGE
Font color: yellow
Background color: red
Correct label: yellow
```

If CLIP predicts `orange`, the model is following the written word instead of the visual font color. This is treated as a **word-bias** error.

### Stage 2: Expanded Prompt Robustness and Multilingual Testing

The second stage extends the original experiment based on supervisor feedback. It adds:

```text
1. prompt robustness testing
2. a standard text-meaning recognition control
3. multilingual / cross-script Stroop testing
4. optional Qwen2.5-VL-3B black-box evaluation
```

The purpose of the expanded stage is to test whether the Stroop-like word-bias effect is:

```text
1. caused only by prompt wording,
2. robust across clearer font/ink-color prompts,
3. dependent on the language/script used in the image,
4. extendable beyond CLIP to newer vision-language models.
```

---

## Repository Contents

The repository currently contains the original CLIP notebook and the expanded experiment notebook.

```text
DissertationStroopEffect/
├── README.md
├── stroop_clip_one_prompt_synthetic_20k_colab.ipynb
├── StroopExpanded.ipynb
├── ResultsForStroopExpanded/
│   ├── clip_multilingual_stroop_predictions.csv
│   ├── clip_multilingual_summary.csv
│   ├── clip_prompt_battery_predictions.csv
│   ├── clip_prompt_robustness_summary.csv
│   ├── clip_text_meaning_accuracy_by_type.csv
│   ├── clip_text_meaning_accuracy_by_word.csv
│   ├── clip_text_meaning_control_predictions.csv
│   ├── cross_script_word_bias_plot.png
│   ├── prompt_robustness_word_bias_plot.png
│   └── updated_project_summary.txt
```

> Note: notebook and folder names may vary slightly depending on the local file name used during upload.

---

# Stage 1: Original CLIP White-Box Testing

## Dataset

The dataset is generated synthetically inside the Colab notebook. No external dataset is required.

### Dataset Size

```text
Total images: 20,000
Color classes: 10
Image size: 224 x 224
```

### Color Classes

```text
red, blue, green, yellow, orange, purple, brown, gray, black, white
```

### Conditions

The dataset contains four conditions:

| Condition | Description | Example |
|---|---|---|
| `congruent` | Written word and font color match | `RED` written in red |
| `incongruent` | Written word and font color conflict | `ORANGE` written in yellow |
| `neutral` | Word is not a color word | `DOG` written in yellow |
| `background_conflict` | Word, font color, and background create competing cues | `ORANGE` written in yellow on red background |

The correct label is always the **visual font color**.

---

## Stage 1 Methodology

### 1. Black-Box Testing

For black-box testing, the model is evaluated only using its final prediction.

The original experiment uses one fixed prompt template:

```text
the text is written in {color}
```

For each image, the notebook creates 10 candidate prompts:

```text
the text is written in red
the text is written in blue
the text is written in green
...
the text is written in white
```

CLIP compares the image with all candidate prompts and selects the color with the highest image-text similarity score.

The prediction is then compared with the true font color.

### Error Categories

Each prediction is categorized as:

| Category | Meaning |
|---|---|
| `correct_font_color` | Prediction matches the actual font color |
| `word_bias` | Prediction matches the written color word |
| `background_bias` | Prediction matches the background color |
| `other_error` | Prediction does not match font color, word color, or background color |

---

### 2. White-Box Testing

White-box testing is used to understand **why** CLIP made a prediction.

For each selected image, the notebook generates a heatmap using one attribution prompt based on CLIP's predicted label.

Example:

```text
CLIP prediction: orange
Attribution prompt: the text is written in orange
```

This means the heatmap explains the model's actual decision.

The heatmap shows which image regions contributed most to the image-text similarity score for the selected prompt.

---

### 3. Layer-Wise White-Box Analysis

The project also includes layer-wise attribution through selected CLIP ViT-B/16 visual transformer layers:

```text
Layer 0, Layer 3, Layer 6, Layer 9
```

This helps analyze how the model's visual evidence changes across the network.

The same one-prompt setup is used:

```text
the text is written in {predicted_color}
```

For each layer, the notebook generates a heatmap and calculates text-region focus metrics using the binary text mask.

### Text Enrichment

The main metric used for layer-wise analysis is **text enrichment**.

Text enrichment measures whether the heatmap is more concentrated on the text region than expected based on the size of the text area.

```text
text_enrichment > 1
```

means the model's attribution is more focused on the text region than expected by area alone.

---

# Stage 2: Expanded Prompt Robustness and Multilingual Testing

The expanded experiment was added after supervisor feedback to test whether the word-bias result is robust and whether it changes across scripts.

## New Experiments Added

### 1. Prompt Robustness Testing

The original prompt:

```text
the text is written in {color}
```

can be ambiguous because it may describe either:

```text
1. the ink/font color of the text
2. the word written in the image
```

To test whether the result is only caused by prompt wording, the expanded notebook evaluates multiple prompt templates.

### Prompt Battery

| Prompt Key | Prompt Template | Purpose |
|---|---|---|
| `ambiguous_written_in` | `the text is written in {color}` | Original ambiguous prompt |
| `ink_color` | `the ink color of the text is {color}` | Clear ink/font-color prompt |
| `font_color` | `the font color is {color}` | Clear font-color prompt |
| `letters_color` | `letters written in {color} color` | Clear letter-color prompt |
| `word_oriented_control` | `a photo of the word "{color}"` | Deliberately word-oriented control |
| `minimal_control` | `{color}` | Minimal control prompt |

For each template, the notebook reports:

```text
1. overall accuracy
2. accuracy by condition
3. incongruent word-bias rate
4. background-conflict word-bias rate
```

---

### 2. Standard Text-Meaning Recognition Control

This experiment checks whether CLIP can recognize the meaning of text in a clean setting.

All images use:

```text
black text on white background
```

The text contains both color words and neutral object words:

```text
red, blue, green, yellow, orange, purple, brown, gray, black, white
cat, dog, ship, tree, house, car, book, phone, chair, music
```

The model is prompted to recognize the textual meaning of the image.

This experiment avoids framing the task only as OCR. Instead, it checks whether the model can recognize written word meaning under a standard condition.

---

### 3. Multilingual / Cross-Script Stroop Testing

The expanded notebook also tests Stroop-style interference across scripts.

Currently included scripts:

```text
English
Chinese
```

Example:

```text
Chinese word: 红色
Semantic meaning: red
Font color: blue
Correct answer: blue
```

If the model predicts `red`, then it is following the written word meaning instead of the visual font color.

The multilingual experiment tests whether word-bias is stronger when the model can process the written script.

---

### 4. Optional Advanced VLM Evaluation

The expanded notebook includes an optional section for testing a newer instruction-following vision-language model:

```text
Qwen/Qwen2.5-VL-3B-Instruct
```

This section is turned off by default because it requires more GPU memory and runtime than CLIP.

It can be enabled inside the notebook by changing:

```python
RUN_QWEN_SETUP = True
RUN_QWEN_EVAL = True
```

The recommended first run is a small sample:

```python
QWEN_SAMPLE_SIZE = 100
```

---

## Models

The main model used in the first stage is:

```text
CLIP ViT-B/16
```

ViT-B/16 is used because it produces a 14 × 14 patch grid, which is useful for patch-level heatmap visualization.

The expanded notebook currently includes:

```text
CLIP ViT-B/16
Optional: Qwen2.5-VL-3B-Instruct
```

Future experiments may include:

```text
CLIP ViT-B/32
CLIP ViT-L/14
OpenCLIP
SigLIP
Qwen2.5-VL 3B / 7B
Gemma 3 vision-capable variants
```

---

## Notebooks

### Original notebook

```text
stroop_clip_one_prompt_synthetic_20k_colab.ipynb
```

This notebook performs:

```text
1. synthetic 20k dataset generation
2. one-prompt CLIP black-box evaluation
3. one-prompt CLIP white-box heatmaps
4. layer-wise CLIP attribution
5. result and summary file generation
```

### Expanded notebook

```text
StroopExpanded.ipynb
```

This notebook performs:

```text
1. synthetic 20k English Stroop dataset generation
2. prompt robustness testing
3. standard text-meaning recognition control
4. English vs Chinese cross-script Stroop testing
5. optional Qwen2.5-VL-3B evaluation
6. result plots and CSV summary generation
```

---

## Output Files

### Stage 1 outputs

After running the original notebook, the following outputs are generated:

```text
/content/stroop_clip_one_prompt_dataset/
├── images/
├── masks/
└── results/
    ├── blackbox_one_prompt_predictions.csv
    ├── accuracy_by_condition.csv
    ├── error_counts_by_condition.csv
    ├── error_proportions_by_condition.csv
    ├── accuracy_by_condition.png
    ├── heatmaps/
    ├── layerwise_heatmaps/
    ├── layerwise_one_prompt_summary.csv
    ├── layerwise_text_enrichment_plot.png
    └── professor_email_summary.txt
```

### Stage 2 outputs

After running the expanded notebook, the following outputs are generated:

```text
/content/stroop_expanded_vlm/
├── main_stroop_20k/
│   ├── images/
│   ├── masks/
│   └── metadata.csv
├── text_meaning_control/
│   ├── images/
│   ├── masks/
│   └── metadata.csv
├── multilingual_stroop/
│   ├── images/
│   ├── masks/
│   └── metadata.csv
└── results/
    ├── clip_prompt_battery_predictions.csv
    ├── clip_prompt_robustness_summary.csv
    ├── prompt_robustness_word_bias_plot.png
    ├── clip_text_meaning_control_predictions.csv
    ├── clip_text_meaning_accuracy_by_type.csv
    ├── clip_text_meaning_accuracy_by_word.csv
    ├── clip_multilingual_stroop_predictions.csv
    ├── clip_multilingual_summary.csv
    ├── cross_script_word_bias_plot.png
    └── updated_project_summary.txt
```

The GitHub repository may include a copied results folder:

```text
ResultsForStroopExpanded/
```

---

## Current Expanded Results

The expanded CLIP experiment produced the following key results.

### Prompt Robustness

Word-bias remains high even with clearer font/ink-color prompts.

| Prompt Key | Overall Accuracy | Incongruent Word Bias | Background-Conflict Word Bias |
|---|---:|---:|---:|
| `ambiguous_written_in` | 0.4451 | 0.9688 | 0.9518 |
| `font_color` | 0.4780 | 0.9276 | 0.9280 |
| `ink_color` | 0.4876 | 0.9050 | 0.8592 |
| `letters_color` | 0.4505 | 0.9894 | 0.9856 |
| `minimal_control` | 0.4358 | 0.9996 | 0.9942 |
| `word_oriented_control` | 0.4355 | 1.0000 | 1.0000 |

This suggests that prompt ambiguity may affect the strength of the bias, but it does not fully explain the effect.

### Text-Meaning Control

```text
Text-meaning control overall accuracy: 1.0000
```

This shows that CLIP can recognize the written English words in a clean black-text setting.

### Multilingual / Cross-Script Result

| Script | Condition | Accuracy | Word-Bias Rate |
|---|---|---:|---:|
| Chinese | background_conflict | 0.3020 | 0.0010 |
| Chinese | congruent | 0.8060 | 0.0000 |
| Chinese | incongruent | 0.7890 | 0.0190 |
| English | background_conflict | 0.0850 | 0.8170 |
| English | congruent | 1.0000 | 0.0000 |
| English | incongruent | 0.1000 | 0.9000 |

The English Stroop images produce strong word bias, while Chinese Stroop images produce very little word bias for CLIP.

---

## Current Interpretation

The original experiment showed that CLIP often predicts the written word instead of the font color in Stroop-style conflict images.

The expanded experiment strengthens this conclusion in two ways:

1. **Prompt robustness:**  
   The word-bias effect remains high even when using clearer prompts such as:

   ```text
   the ink color of the text is {color}
   the font color is {color}
   ```

   This suggests the effect is not only caused by the ambiguous wording of the original prompt.

2. **Cross-script behavior:**  
   CLIP shows strong word bias for English text, but almost no word bias for Chinese text.

   This supports the interpretation that Stroop-like interference depends on whether the model can process the written word meaning in that script.

Overall, the current result suggests:

> CLIP often uses the text region as evidence, but under visual-semantic conflict, it aligns that region more strongly with the written word than with the actual font color.

The black-box results show **what** CLIP predicted.

The white-box heatmaps help explain **where** the model looked.

The layer-wise analysis helps show **how the evidence changes across visual transformer layers**.

The prompt-robustness and multilingual experiments help test whether the effect is robust and whether it depends on script-level text understanding.

---

## How to Run

### Step 1: Open the notebook in Google Colab

For the original CLIP white-box experiment, open:

```text
stroop_clip_one_prompt_synthetic_20k_colab.ipynb
```

For the expanded experiment, open:

```text
StroopExpanded.ipynb
```

### Step 2: Enable GPU

In Colab:

```text
Runtime → Change runtime type → Hardware accelerator → GPU
```

Then verify:

```python
import torch
print(torch.cuda.is_available())
```

### Step 3: Run all cells

The notebooks will automatically:

```text
1. generate synthetic datasets
2. run model evaluation
3. calculate metrics
4. generate plots
5. save result files
```

### Step 4: Review outputs

Important Stage 1 outputs:

```text
accuracy_by_condition.csv
error_proportions_by_condition.csv
one_prompt_heatmap_metrics_word_bias.csv
layerwise_one_prompt_summary.csv
layerwise_text_enrichment_plot.png
professor_email_summary.txt
```

Important Stage 2 outputs:

```text
clip_prompt_robustness_summary.csv
prompt_robustness_word_bias_plot.png
clip_text_meaning_accuracy_by_type.csv
clip_text_meaning_accuracy_by_word.csv
clip_multilingual_summary.csv
cross_script_word_bias_plot.png
updated_project_summary.txt
```

---

## Future Work

Possible extensions include:

- testing more CLIP variants such as ViT-B/32 and ViT-L/14,
- testing OpenCLIP and SigLIP,
- extending the black-box evaluation to Qwen2.5-VL 3B/7B,
- testing Gemma 3 vision-capable variants,
- comparing the custom attribution method with official Grad-ECLIP,
- adding text-encoder attribution,
- increasing dataset realism using more fonts, rotations, blur, shadows, noise, and natural backgrounds,
- expanding the multilingual Stroop test to more scripts such as Arabic, Korean, Greek, and Cyrillic,
- fine-tuning CLIP on Stroop-style conflict data to test whether word bias can be reduced.

---

## Project Status

This is a first-draft research implementation.

The repository currently contains:

```text
Stage 1:
Synthetic dataset generation
One-prompt CLIP evaluation
One-prompt white-box heatmaps
Layer-wise attribution analysis

Stage 2:
Prompt robustness testing
Text-meaning recognition control
English-Chinese cross-script Stroop testing
Optional Qwen2.5-VL evaluation setup
```

Further experiments will be added based on supervisor feedback.

---

## Author

```text
Aishwarya Vijay
```
