# Z-Image Turbo

Fast standing model on the [shared Comfy install](comfy-omarchy.md). Eight steps. Apache-2.0. Measured 2026-10-11.

Weights: [Comfy-Org/z_image_turbo](https://huggingface.co/Comfy-Org/z_image_turbo), split files. Model card: [Tongyi-MAI/Z-Image-Turbo](https://huggingface.co/Tongyi-MAI/Z-Image-Turbo).

| File | Size we downloaded | Comfy directory |
|------|--------------------|-----------------|
| `split_files/diffusion_models/z_image_turbo_bf16.safetensors` | 12G | `models/diffusion_models/` |
| `split_files/text_encoders/qwen_3_4b.safetensors` | 7.5G | `models/text_encoders/` |
| `split_files/vae/ae.safetensors` | 320M | `models/vae/` |

```bash
uvx --from huggingface_hub hf download Comfy-Org/z_image_turbo \
  --include 'split_files/diffusion_models/*.safetensors' \
  --include 'split_files/text_encoders/*.safetensors' \
  --include 'split_files/vae/*.safetensors' \
  --local-dir /mnt/models/comfy/z-image/hf

cd ~/comfyui-zimage
ln -sfn /mnt/models/comfy/z-image/hf/split_files/diffusion_models/z_image_turbo_bf16.safetensors models/diffusion_models/
ln -sfn /mnt/models/comfy/z-image/hf/split_files/text_encoders/qwen_3_4b.safetensors models/text_encoders/
ln -sfn /mnt/models/comfy/z-image/hf/split_files/vae/ae.safetensors models/vae/
```

Use whatever disk you have. `/mnt/models` is where we put them. The symlink is the part Comfy cares about.

## Graph

Upstream template, same checkout family: [Text to Image (Z-Image-Turbo)](https://github.com/comfyanonymous/ComfyUI/blob/master/blueprints/Text%20to%20Image%20(Z-Image-Turbo).json). Example page: [ComfyUI Z-Image](https://comfyanonymous.github.io/ComfyUI_examples/z_image/).

Settings that produced the smoke and the eight-image set:

- `UNETLoader` on `z_image_turbo_bf16.safetensors`
- `CLIPLoader` type `lumina2` on `qwen_3_4b.safetensors`
- `VAELoader` on `ae.safetensors`
- `ConditioningZeroOut` for the negative
- `EmptySD3LatentImage`, 1024×1024
- `ModelSamplingAuraFlow` shift `3`
- `KSampler`: `res_multistep`, `simple`, cfg `1`, `8` steps

Do not reuse the Qwen-Image-2.1 sampler here. That one is euler, 25 steps, and it does not use the AuraFlow shift.

## What we measured

1024 smoke, prompt "a red ceramic mug on a wooden table, soft window light". Finished. Edge 43°C. About 21 GB resident.

Eight 1024 images after that, one job at a time, peaked at 56°C. Readable Latin on a short sign. Chinese on a lantern was correct. A second stall sign was junk. Hands were weak. That is the capability we saw, not a leaderboard claim.

If Qwen-Image-2.1 is already loaded, restart the unit first. See [Heat](thermal.md) if you are about to run a long job on a warm box. Eight-step Z-Image did not need the 120-second hold. Twenty-five-step Qwen did.
