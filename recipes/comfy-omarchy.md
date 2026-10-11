# ComfyUI on Omarchy, gfx1151

This is the shared stack for both image recipes. Measured on a GMKtec EVO-X2, Omarchy 4.0.4, kernel `7.2.5-3-omarchy`, Mesa `1:26.2.2-1`. ComfyUI `0.39.0`, git `7f7fd91` (2026-10-10).

You need `/dev/kfd`, a Vulkan Radeon stack, and enough disk for about 30 GB of weights if you keep both models. Python 3.12. We used `uv`.

## Do not install the generic ROCm wheel

This failed:

```bash
# do not run
pip install torch==2.11.0+rocm7.1 --index-url https://download.pytorch.org/whl/rocm7.1
```

The GPU showed up. The first kernel died. The process segfaulted in `libhsa-runtime64.so`. Setting `HSA_OVERRIDE_GFX_VERSION=11.0.0` did not fix it.

The wheel that worked came from AMD's gfx1151 extra, not from that index.

## Install

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
git clone https://github.com/comfyanonymous/ComfyUI.git ~/comfyui-zimage
cd ~/comfyui-zimage
git checkout 7f7fd91

uv venv --python 3.12 .venv
uv pip install --python .venv/bin/python \
  --index-url https://stable.repo.amd.com/rocm/whl-next/ \
  --prerelease=allow \
  'torch[device-gfx1151]' 'torchvision[device-gfx1151]'
```

Check the result before you install Comfy's requirements. Ours printed:

```text
torch 2.14.0+rocm10.1.0
torchvision 0.29.0a0+rocm10.1.0
device name Radeon 8060S
```

`0.29` is a prerelease. Without `--prerelease=allow`, uv will not take it, and Comfy will not stay up on a mismatched torchvision.

Then install Comfy's other dependencies without letting that file replace torch:

```bash
grep -viE '^(torch|torchvision|torchaudio)([=<>~ ]|$)' requirements.txt > /tmp/comfy-reqs.txt
uv pip install --python .venv/bin/python -r /tmp/comfy-reqs.txt
```

Smoke the GPU before you start the server:

```bash
.venv/bin/python - << 'PY'
import torch
print(torch.__version__)
print(torch.cuda.get_device_name(0))
print(torch.zeros(8, device="cuda"))
PY
```

If that allocation segfaults, stop. Do not paper over it with `HSA_OVERRIDE_GFX_VERSION`.

## Service

Bind localhost. Replace `USER` in [the unit file](../templates/comfyui-zimage.service), then:

```bash
mkdir -p ~/.config/systemd/user
cp templates/comfyui-zimage.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now comfyui-zimage.service
curl -sf http://127.0.0.1:8188/system_stats
```

A ready server returned ComfyUI `0.39.0` and pytorch `2.14.0+rocm10.1.0`.

Logs go to `~/comfyui-zimage/comfy.log`. The unit restarts on failure after 15 seconds. It does not survive a full logout unless linger is enabled:

```bash
sudo loginctl enable-linger "$USER"
```

We could not run that without a password. Until you do, the server lives with the user session.

## One model at a time

`POST /free` did not unload the previous weights. About 20 GB stayed resident. Restart the unit before the first load of the other model:

```bash
systemctl --user restart comfyui-zimage.service
```

After that restart, before the Qwen smoke, VRAM was 1.19 GiB and the edge was 41°C.
