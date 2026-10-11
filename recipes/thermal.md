# Heat

The model was not the failure. The second job was.

On this EVO-X2 the useful sensor is the amdgpu edge diode:

```bash
cat /sys/class/drm/card1/device/hwmon/hwmon*/temp1_input
```

Divide by 1000. That is degrees C. There was no fan RPM in that hwmon, and no power cap. Software cannot tell you whether the chassis fan is keeping up. Watch the edge, the power input if it is present (`power1_input`, microwatts), and `pp_dpm_sclk`.

Idle after a real cool-down: 41°C, 13.6 W, 600 MHz. Under a job the clock table goes to 2900 MHz.

## What happened

| Job | Steps | Start | Result |
|-----|-------|-------|--------|
| Z-Image smoke | 8 | cold | finished, 43°C, ~21 GB resident |
| Z-Image, eight images | 8 | one at a time | finished, peak 56°C |
| Qwen smoke | 25 | after a unit restart, 41°C | finished, peak 81°C, ~16 GiB resident |
| Qwen, next job | 25 | edge still 46–50°C | 92–94°C within seconds, stopped |

Die temperature is not the sink. A 46°C reading after a 70-second denoise does not mean the chassis is ready for another 25-step job. That is why the second job spikes and the first one does not.

## Rule that finished the set

Before each 25-step Qwen job:

1. Wait until the edge is at or under 46°C.
2. Hold that for 120 seconds. If it climbs, the hold restarts.
3. Then submit one job.

While it runs, sample every 5 seconds.

- One reading in the mid-80s is the opening spike. The smoke finished through 81°C. Do not kill the job for that alone.
- Stop the job if the edge reaches 92°C, or if it stays at or above 86°C for three polls in a row.

Interrupt with `POST /interrupt`, then wait for the hold again. Do not queue the next prompt behind a hot one.

Eight-step Z-Image did not need this hold. Twenty-five-step Qwen did. If you add a 2048 job, treat it as hotter than the 1024 case until you have measured it. We have not.
