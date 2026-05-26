# ExpressEdit: Fast Editing of Stylized Facial Expressions with Diffusion Models in Photoshop

[[Paper]](https://arxiv.org/abs/2604.03448) [[Website]](https://kenantang.github.io/ExpressEdit/) [[Dataset]](https://huggingface.co/datasets/JiashengGuo/ExpressEdit) [[RAG Demo]](https://huggingface.co/spaces/JiashengGuo/ExpressEdit-RAG)

## News

[06.04.2026] ExpressEdit is presented at the CVPR 2026 Workshop on AI for Visual Arts.

[04.03.2026] Paper and code released on arXiv.

[02.2026] ExpressEdit Photoshop plugin released.

## Introduction

ExpressEdit is a free, open-source Photoshop plugin for editing stylized facial expressions on illustrated characters. It uses the [SPICE](https://github.com/kenantang/spice) image editing workflow as its backend and supports 135 expression tags from Danbooru.

Key features:

- **No global noise.** Only the selected region is modified. No watermarks, no pixel drift.
- **Precise control.** Native Photoshop operations (Scale, Liquify, Brush) serve as hints to the diffusion model, enabling pixel-level precision that text prompts alone cannot achieve.
- **Fast.** Under 3 seconds per edit on a single consumer-grade GPU, with zero API cost.
- **135 expression tags** with definitions, alternative tags, example images, and multilingual stories for retrieval-augmented generation.

## Installation

ExpressEdit has two components: the Photoshop plugin (frontend) and the SPICE backend. Both need to be running for the plugin to work.

### Backend Setup

The backend uses [SPICE](https://github.com/kenantang/spice) with the following models:

- **Base model:** [WAI-illustrious-SDXL](https://civitai.com/models/827184?modelVersionId=2514310)
- **ControlNet:** [diffusers_xl_canny_mid](https://huggingface.co/lllyasviel/sd_control_collection/blob/main/diffusers_xl_canny_mid.safetensors) (Canny edge ControlNet for SDXL)
- **Speed-up LoRA (optional):** [SDXL Lightning LoRA (8 steps)](https://civitai.com/models/350450?modelVersionId=391999)

Follow the [SPICE installation guide](https://github.com/kenantang/spice#installation) to set up the backend in Stable Diffusion Web UI (Automatic1111 or Forge). The SPICE backend has been used by the community for over two years, and there are many existing guides for setting it up on different systems.

After setting up the backend, make sure the API is enabled and accessible.

### Plugin Setup

1. Install [Adobe UXP Developer Tool](https://developer.adobe.com/photoshop/uxp/2022/guides/devtool/).
2. Load the ExpressEdit plugin from the `plugin/` folder in this repository.
3. In the plugin settings, point the API endpoint to your running SPICE backend.

The plugin was developed and tested on Photoshop version 27.4.0. Newer versions should also work. If you encounter compatibility issues, please open a GitHub issue.

### System Requirements

- Adobe Photoshop (v27.4.0 or later recommended)
- An NVIDIA GPU with sufficient VRAM to run the SDXL base model and ControlNet (tested on RTX 4090 and RTX A6000)
- Python environment for the SPICE backend

If you have trouble setting up, feel free to [open an issue](https://github.com/kenantang/ExpressEdit/issues) or contact the authors.

## Usage

### Basic Workflow

1. **Set the prompt.** Enter an expression tag (e.g. `smile`, `clenched teeth`, `+_+`) in the plugin prompt box. The tag is inserted between a prefix (character description) and a suffix (style keywords) to form the full prompt.

2. **Apply a transformation (optional).** Use native Photoshop operations to roughly hint at the desired expression:
   - **Scale** to resize facial elements (e.g. shrink the irises for a surprised look)
   - **Liquify** to drag elements to new positions (e.g. move eyes to one side for an averted gaze)
   - **Brush** to sketch rough shapes (e.g. paint a simple open mouth shape)
   
   100 out of 135 expression tags can be edited without any transformation.

3. **Select the region.** Use Quick Selection or the Selection Brush to cover the area to be edited. The selection does not need to be precise. For edits involving only the eyes or mouth, add context dots around the selection for better results.

4. **Click Generate.** The result appears as a new layer. Merge it when satisfied.

### Prompt Format

ExpressEdit uses a tag-based prompt format (not natural language). A typical prompt looks like:

```
1girl, character_name, white background, smile, masterpiece, best quality, amazing quality
```

The expression tag (`smile`) is inserted between the prefix and suffix. You can combine multiple tags (e.g. `smile, blush`) for richer expressions.

If you are not familiar with the tags, use the [RAG demo](https://huggingface.co/spaces/JiashengGuo/ExpressEdit-RAG) to find the right tag from a story description.

## Tips

**Model version consistency.** The base model used for generating the original image and for editing should be the same version. Switching model versions (e.g. WAI-illustrious v16 vs v17) can cause style mismatches such as different default lighting or facial proportions.

**Denoising strength.** The default 70% works well in most cases. For edits involving hair or fine details, reducing to around 50% can avoid artifacts on weaker edges that may not be captured by the Canny edge ControlNet.

**Selection size.** Larger selections generally produce better results. When the selection is too small, the model operates on limited context and may produce edge mismatches or dimensional squeezing.

**Context dots.** When editing only a sub-region of the face (just the eyes or mouth), adding context dots around the selection helps the model understand the full facial layout and produce more coherent results.

**Speed-up LoRA.** The Lightning LoRA reduces inference from approximately 4 seconds to approximately 2 seconds. Use it for rapid prototyping. Use the full 30-step model for final renders, as it produces higher quality details (especially on eyelashes and fine lines).

**Transformation-free tags.** 100 out of 135 expression tags can be edited without any Photoshop transformation. The remaining 35 tags (flagged in the plugin documentation) require a quick Liquify or Scale step. Examples of transformation-required tags include `averting eyes`, where the irises need to be physically moved to the desired position.

**Combining eye and mouth edits.** For complex expressions, edit the eyes and mouth separately with different tags and selections. This gives more control than using a single combined tag.

**Repairing artifacts.** If minor artifacts appear around the selection edge after many edits, simply select the noisy region and generate again with the same prompt. The noise is removed in one step.

## Expression Database

The dataset is available at [huggingface.co/datasets/JiashengGuo/ExpressEdit](https://huggingface.co/datasets/JiashengGuo/ExpressEdit).

Each of the 135 expression tags includes:

- **Definition** from the official Danbooru wiki
- **Alternative tags** in Japanese, Chinese, and Korean (from Pixiv)
- **Example images** (5 per tag, across 5 different characters, 3375 total)
- **Example stories** (5 per tag in each of English, Chinese, Japanese, and Korean, 2700 total)
- **Transformation-free editing flag** indicating whether the tag can be edited without Photoshop transformations

## Applications

ExpressEdit is designed for professional and semi-professional illustration workflows:

- **Character design and iteration.** Quickly explore diverse stylized expressions on a character, taking advantage of the clean, noise-free editing that preserves character consistency across iterations.
- **Storyboard visualization.** Match character expressions to narrative beats using the RAG system, which retrieves the right expression tag from a story description in four languages.
- **Comic and manga production.** Edit expressions across panels efficiently with simple sketches and selections, without advanced digital painting knowledge.
- **Fixing AI-generated artifacts.** Repair incorrect details on AI-generated characters (e.g. wrong number of accessories, scrambled colors) using the Brush and Select workflow.

## Citation

```bibtex
@article{tang2026expressedit,
  title   = {ExpressEdit: Fast Editing of Stylized Facial
             Expressions with Diffusion Models in Photoshop},
  author  = {Tang, Kenan and Guo, Jiasheng and Lin, Jeffrey
             and Qin, Yao},
  journal = {arXiv preprint arXiv:2604.03448},
  year    = {2026}
}
```
