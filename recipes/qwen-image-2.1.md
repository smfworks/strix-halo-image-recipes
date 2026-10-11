# Qwen-Image-2.1

Quality model on the same Comfy process as [Z-Image Turbo](z-image-turbo.md). Not a second server. Measured 2026-10-11.

Read the license before you redistribute anything. [Qwen/Qwen-Image-2.1](https://huggingface.co/Qwen/Qwen-Image-2.1) is not Apache-2.0. The Hugging Face license field is `other`.

We used the INT8 trio from [Comfy-Org/Qwen-Image-2.1](https://huggingface.co/Comfy-Org/Qwen-Image-2.1), not the BF16 pair. The INT8 set is about 16 GB of weights. It fit the 48 GiB carveout with the desktop up. We did not download the prompt-enhancer.

| File | Size we downloaded | Comfy directory |
|------|--------------------|-----------------|
| `qwen_image_2.1_int8_convrot.safetensors` | 6.8G | `models/diffusion_models/` |
| `qwen3vl_8b_int8_convrot.safetensors` | 8.8G | `models/text_encoders/` |
| `qwen_image_2.1_vae_bf16.safetensors` | 645M | `models/vae/` |

Comfy 0.39.0 already had `TextEncodeQwenImage21` and `QwenImage21Cache`. CLIPLoader type includes `qwen_image`. The official workflow JSON was not in the installed template package, so we submitted the API graph. Upstream writeup: [Qwen-Image-2.1 in Comfy](https://docs.comfy.org/tutorials/image/qwen/qwen-image-2-1).

## Load it

Restart the unit if Z-Image is resident. `POST /free` left about 20 GB.

```bash
systemctl --user restart comfyui-zimage.service
```

Symlink the three files into `models/diffusion_models`, `models/text_encoders`, and `models/vae`. Same pattern as the Z-Image guide.

## Graph

- `UNETLoader` on `qwen_image_2.1_int8_convrot.safetensors`, weight dtype default
- `CLIPLoader` type `qwen_image` on `qwen3vl_8b_int8_convrot.safetensors`
- `VAELoader` on `qwen_image_2.1_vae_bf16.safetensors`
- `QwenImage21Cache`, device `auto`, dtype `default`
- `TextEncodeQwenImage21` at resolution 1024. It returns positive, negative, and the latent. You do not add `EmptySD3LatentImage`.
- `KSampler`: `euler`, `simple`, cfg `1`, `25` steps
- No `ModelSamplingAuraFlow`. That shift belongs to Z-Image.

## What we measured

1024 smoke, rainy night, neon sign reading `QWEN IMAGE 2.1`. The sign came back as QWEN / IMAGE 2.1. Peak edge 81°C, then 52°C. About 16 GiB stayed resident after the job.

The same eight prompts used for Z-Image also finished, one job at a time, after the cool-down in [Heat](thermal.md). Early attempts started on a warm sink and were stopped at 92–94°C. Those files are not the set.

We did not run 2048. The 1024 smoke already reached 81°C. Do that only after you have watched the edge on a 1024 job.
