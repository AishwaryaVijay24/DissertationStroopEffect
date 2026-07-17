


# Stroop-Style CLIP White-Box Testing

This project investigates whether CLIP follows the **actual visual font color** in Stroop-style images or whether it becomes biased toward the **written word**.

The experiment is inspired by the human Stroop effect, where the meaning of a written color word can interfere with naming the ink color. In this project, the correct label is always the **font color**, but the image may contain conflicting cues such as the written word and background color.

---

## Project Overview

This project performs both **black-box** and **white-box** testing on CLIP.

### Main Research Question

> When the written word and the visual font color conflict, does CLIP predict the actual font color or follow the written word?

Example:

```text
Word shown: ORANGE
Font color: yellow
Background color: red
Correct label: yellow
```

If CLIP predicts `orange`, the model is following the written word instead of the visual font color. This is treated as a **word-bias** error.

---

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

## Methodology

### 1. Black-Box Testing

For black-box testing, the model is evaluated only using its final prediction.

The experiment uses one fixed prompt template:

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

## Model

The main model used in this project is:

```text
CLIP ViT-B/16
```

ViT-B/16 is used because it produces a 14 × 14 patch grid, which is useful for patch-level heatmap visualization.

---

## Notebook

The main notebook is:

```text
stroop_clip_one_prompt_synthetic_20k_colab.ipynb
```

The notebook performs the following steps:

1. Installs CLIP and dependencies.
2. Generates the synthetic Stroop-style dataset.
3. Saves images, masks, and metadata.
4. Loads CLIP ViT-B/16.
5. Runs one-prompt black-box evaluation.
6. Calculates accuracy and error categories.
7. Generates one-prompt white-box heatmaps.
8. Performs layer-wise white-box analysis.
9. Saves results and summary files.

---

## Output Files

After running the notebook, the following outputs are generated:

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

---

## How to Run

### Step 1: Open the notebook in Google Colab

Upload or open:

```text
stroop_clip_one_prompt_synthetic_20k_colab.ipynb
```

### Step 2: Run all cells

The notebook will automatically:

- generate the dataset,
- evaluate CLIP,
- calculate results,
- generate heatmaps,
- save outputs.

### Step 3: Review results

Important outputs to check:

```text
accuracy_by_condition.csv
error_proportions_by_condition.csv
one_prompt_heatmap_metrics_word_bias.csv
layerwise_one_prompt_summary.csv
layerwise_text_enrichment_plot.png
professor_email_summary.txt
```

---

## Current Interpretation

The expected interpretation of the experiment is:

> CLIP often focuses on the text region, but under Stroop-style conflict, its prediction may align more strongly with the written word than with the actual visual font color.

This suggests that CLIP can show a **Stroop-like word bias**.

The black-box results show **what** CLIP predicted.

The white-box heatmaps help explain **where** the model looked.

The layer-wise analysis helps show **how the evidence changes across visual transformer layers**.

---

## Future Work

Possible extensions include:

- testing more CLIP variants such as ViT-L/14,
- testing OpenCLIP and other vision-language models,
- comparing the custom attribution method with official Grad-ECLIP,
- adding text-encoder attribution,
- increasing dataset realism using more fonts, rotations, blur, shadows, noise, and natural backgrounds,
- testing black-box multimodal models such as ChatGPT or Claude,
- fine-tuning CLIP on Stroop-style conflict data to test whether word bias can be reduced.

---

## Project Status

This is a first-draft research implementation. The current version focuses on:

```text
Synthetic dataset generation
One-prompt CLIP evaluation
One-prompt white-box heatmaps
Layer-wise attribution analysis
```

Further experiments can be added based on supervisor feedback.

---

## Author

```text
Aishwarya Vijay
```
