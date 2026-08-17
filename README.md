# Multilingual Stroop Benchmark for Vision-Language Models

This repository contains the code, results, and paper artefacts for the dissertation project:

**A Multilingual Stroop Benchmark for Testing Visual-Textual Cue Interference in Vision-Language Models**

The project studies whether vision-language models identify the **visible font color** in Stroop-style images, or whether their predictions are influenced by the **written word meaning** or **background color**.

---

## Project Summary

In every Stroop-style image, the correct answer is always the **font color**.

Example:

```text
Word shown: RED
Font color: yellow
Background color: white
Correct answer: yellow
```

If a model predicts `red`, it is following the written word meaning instead of the visible font color. This is treated as a **word-bias error**.

---

## Models Evaluated

The project evaluates:

- **CLIP ViT-B/16**
- **Qwen2.5-VL-3B-Instruct**

CLIP is evaluated using zero-shot image-text similarity.  
Qwen2.5-VL is evaluated using direct visual-question prompts.

---

## Experiments Included

The repository supports the experiments reported in the dissertation paper:

1. **Main English Stroop evaluation**  
   Tests CLIP on English Stroop-style images.

2. **Prompt robustness evaluation**  
   Tests whether CLIP word bias remains across vague, exact, descriptive, and control prompts.

3. **Text-meaning recognition control**  
   Checks whether CLIP can recognize written word meaning in clean black-text images.

4. **Multilingual Stroop evaluation**  
   Tests script-dependent cue interference across English, Chinese, Hindi, and Arabic.

5. **Qwen2.5-VL evaluation**  
   Evaluates an instruction-following VLM on a balanced multilingual subset.

6. **Interpretability outputs**  
   Includes CLIP Grad-CAM-style heatmaps and Qwen attention-based visual token heatmaps.

---

## Dataset Overview

The full benchmark contains synthetic images generated in the Colab notebook.

| Experiment | Description |
|---|---|
| English Stroop dataset | Font-color recognition under English cue conflict |
| Prompt robustness dataset | Same images evaluated with multiple prompt templates |
| Text-meaning control | Clean word-recognition control using black text |
| Multilingual Stroop dataset | English, Chinese, Hindi, and Arabic Stroop-style images |
| Qwen subset | Balanced multilingual subset for instruction-following VLM evaluation |

The color label space is:

```text
red, blue, green, yellow, orange, purple, brown, gray, black, white
```

---

## Repository Contents

```text
DissertationStroopEffect/
├── README.md
├── .gitignore
├── Stroop_VLM_Benchmark_Colab.ipynb
├── Paper Draft and Final/
│   ├── Dissertation Draft1.pdf
│   └── Dissertation Final Draft Before Approval.pdf
└── StroopEffectResults/
```

Folder names may vary slightly depending on how the Colab outputs are exported.

---

## How to Run

1. Open the Colab notebook:

```text
Expanded_Multilingual_Stroop_VLM_Benchmark_Colab.ipynb
```

2. Enable GPU in Google Colab:

```text
Runtime → Change runtime type → Hardware accelerator → GPU
```

3. Run the notebook cells to generate datasets, evaluate models, and export results.

---

## Main Outputs

The notebook produces:

- generated image datasets and metadata files
- CLIP prediction and summary CSV files
- Qwen2.5-VL prediction and summary CSV files
- prompt robustness plots
- multilingual word-bias plots
- CLIP heatmaps
- Qwen attention heatmaps

These artefacts support the results reported in the dissertation paper.

---

## Author

**Aishwarya Vijay**  
MSc Artificial Intelligence  
Queen Mary University of London
