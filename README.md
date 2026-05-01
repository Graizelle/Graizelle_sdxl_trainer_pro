# Graizelle's sdxl_trainer_pro
Advanced SDXL trainer notebook optimized for Illustrious 2.0 
## Overview

This notebook is designed as a complete SDXL LoRA training workflow built around Hugging Face repositories using HF Xet storage 10x faster than G Drive or Git LFS,  or manually uploading datasets to Colab each session. your images, captions, trained LoRAs, and processed datasets are managed directly through Hugging Face repos. The notebook automatically downloads your dataset, prepares it for training, performs optional tagging and cleanup, trains your LoRA using `kohya-ss/sd-scripts`, and can upload finished `.safetensors` files back to your Hugging Face LoRA repository.

The workflow is optimized for memory efficiency on free Google Colab GPU runtimes — especially T4 instances — uses latent caching, gradient checkpointing, low RAM optimizations, SDPA attention, background training support, and conservative default batch sizing. The default configuration is intended to provide stable SDXL LoRA training within typical free-tier VRAM limits while still supporting high-quality 768px training workflows. Users with larger GPUs such as L4 or A100 instances can increase batch size, resolution flexibility, worker counts, and network dimensions more aggressively if desired.

Before starting, create a free Hugging Face account if you do not already have one. You will generally want:

- A **Dataset Repo** containing your training images
- An empty **Model/LoRA Repo** where trained LoRAs will be uploaded
- A Hugging Face access token with read/write permissions

After creating your repos, fill out the variables in the configuration sections of the notebook:

- Dataset repo name
- Base SDXL checkpoint repo
- Output LoRA repo
- Trigger token
- Training parameters

Once configured, run the notebook cells sequentially from top to bottom. The environment setup cells install dependencies, clone `sd-scripts`, authenticate with Hugging Face, and prepare your dataset automatically. After setup is complete, launch training using the main training cell. The notebook supports running training in the background so the Colab session remains usable while training progresses.

After starting training:

1. Run the training cell with:
    
    ```
    RUN_TRAINING_IN_BACKGROUND=True
    ```
    
2. Then run the live log viewer cell:
    
    ```
    tail-f /content/training.log
    ```
    

This allows you to monitor loss, step count, saving intervals, and training progress in real time without blocking the notebook interface.

---

## Dataset Preparation & Tagging

The notebook includes a full dataset preparation pipeline designed for SDXL LoRA workflows.

### Dataset Download & Organization

The dataset section downloads your Hugging Face dataset repo and automatically restructures it into the folder layout expected by `sd-scripts`. The notebook also supports repeat-folder naming conventions used for LoRA training.

---

### Duplicate Removal

The duplicate remover scans your dataset for exact duplicate images using file hashing and removes redundant files automatically. This helps improve dataset quality and prevents overtraining on repeated images.

---

### WD14 Auto-Tagging

The WD14 tagging section uses the `SmilingWolf/wd-vit-tagger-v3` model to generate captions for your images automatically.

The tagger:

- analyzes each image
- predicts descriptive tags
- creates `.txt` caption files beside each image
- prepends your trigger token automatically

You can control:

- tag confidence threshold
- whether existing captions are overwritten
- trigger token injection

This is especially useful when preparing large anime, stylized, or booru-style datasets quickly.

---

### Advanced Tag Editing

The advanced tag editor lets you bulk-process caption files after tagging.

You can:

- search/replace tags globally
- remove unwanted tags
- append additional tags to every caption

Common uses include:

- removing low-quality tags
- removing conflicting gender tags
- standardizing character names
- adding style descriptors globally

Example:

```
REMOVE_TAGS="1boy, 1girl"
ADD_TAGS="cinematic lighting, detailed shading"
```

---

### Tag Analysis

The analysis section scans all captions and generates:

- tag frequency statistics
- trigger token validation
- training cycle estimates
- optional tag frequency charts

This helps identify:

- overrepresented tags
- missing trigger tokens
- inconsistent captioning
- dataset imbalance

Proper tagging and dataset cleanup significantly improve LoRA quality, especially for character consistency and style adherence.
