# Experiment card

Date: **2026-11-08**. Task: right-arm artificial-banana pick-and-place.

## Data

50 successful operator-labelled demonstrations were collected on the lab's teleoperation platform and converted to LeRobot format.

| Split | Episodes | Frames |
|---|---:|---:|
| TRAIN | 40 | 4,672 |
| VAL | 5 | 649 |
| TEST | 5 | 658 |

The split is at episode level within one collection session. It does not establish cross-day, cross-camera, or broad scene generalization. Normalizer statistics come only from TRAIN. TEST was not used to select checkpoints or tune hyperparameters.

## Policy and fine-tuning

Base: [`lerobot/pi05_base`](https://huggingface.co/lerobot/pi05_base), revision `22c30e5b34f9e7ab738b1b34fc2fc5ecfe2d28f0`.

Base weight identity: 14,467,165,872 bytes; SHA256 `0eb11ca9587678c1d2ef8cf32807c29f8ce53a2bfdfc1aa4a4c96f16fca59b0f`.

The experiment used a matching lab LeRobot fork and saved processors. The upstream base and the task-specific update were verified before offline deployment.

| Setting | Expert-only | LoRA |
|---|---:|---:|
| Selected checkpoint | 4,000 updates | 2,500 updates |
| Batch size | 4 | 8 |
| Learning rate | 2.5e-5 | 1e-4 |
| Seed | 42 | 42 |
| Warmup updates | 100 | 100 |
| Training examples seen at selected checkpoint | 16,000 | 20,000 |
| Equivalent TRAIN-frame exposures | 3.42 | 4.28 |
| Configured trainable parameter count | about 693.4M | 1,287,168 |
| Adapter configuration | action expert / projections; VLM frozen | r=16, alpha=16; 38 target modules |
| Trainable parameter dtype | FP32 | FP32 |
| Activation precision | BF16 autocast | BF16 autocast |
| Selected VAL loss | 0.27025 | 0.29737 |

The configured parameter count is a named-parameter count; it does not imply that every tensor receives a nonzero gradient on every batch. Checkpoints were selected by VAL rather than assuming that the last checkpoint was best. The two methods have different optimization settings and sample exposure; this is a practical task study rather than a controlled claim about the methods in general.

Training environment: one AMD gfx1100 GPU with 47.98 GiB exposed VRAM, ROCm 7.2, PyTorch 2.9.1. The exact retail GPU model was not independently established.

## Deployment measurements

Deployment environment: NVIDIA GeForce RTX 3060, 12 GB class; PyTorch 2.7.1 + CUDA 11.8 build. The original AMD environment and weights were preserved separately during training; deployment used the selected task update with the same fixed base and saved processors.

Both selected checkpoints completed strict local loading and finite `[1, 50, 14]` action-chunk forward passes on a saved TRAIN observation.

| Saved-observation diagnostic | Expert-only | LoRA |
|---|---:|---:|
| Mean of 5 warm forward timings | 0.496 s | 0.567 s |
| Peak GPU reserved memory | 10.41 GiB | 9.20 GiB |

These timings measure an entire action chunk in that diagnostic, not one executed action. Live execution used synchronous inference, with 10 Hz command spacing inside short chunks and pauses for inference. The overall observed command rate was approximately 4.8 Hz; continuous 10 Hz execution is not claimed.

## Onsite observations

| Checkpoint | Operator-reported successes / attempts |
|---|---:|
| Expert-only, step 4,000 | **9 / 10** |
| LoRA, step 2,500 | **0 / 8** |

The task completion criterion was pick up the banana, place it in the blue plate, release, and lift clear. Success was observed by the onsite operator; software exit status alone was not used as a success label.

The session included executor debugging, different starting joint poses, and intentional object interventions. A one-to-one mapping of all reported attempts to a single frozen protocol has not been established. The counts are reported as a pilot-session result.

Qualitative observations reported by the operator:

- Expert-only handled some small banana-position changes.
- After placement, manual removal of the banana was followed by renewed grasp attempts. There was insufficient time to evaluate completion; whether the limit was the script duration or onsite time was not specified.
- An intentional move to a location absent from the demonstrations failed. This should be analysed separately from small local variations; its inclusion in the aggregate count is not established.

These observations motivate controlled perturbation tests. They do not establish broad generalization or a completed recovery capability.

## Failure and interface notes

The tested LoRA checkpoint showed larger initial-action offsets in a saved-observation comparison and consecutive-target jumps in some live trials. Several starts also fell outside the joint ranges covered by demonstration starts. These are diagnostic observations, not a complete causal explanation of the 0/8 result.

Executor integration changes during the session included handling feedback lag separately from consecutive command jumps, confirming stopping with multiple feedback samples, and matching the existing gripper saturation mapping to `[0,1]`. Invalid/nonfinite values and invalid measured feedback remained grounds to stop. Joint and gripper units are handled separately.

The initial release provides the experiment report and supplied demo media. Raw demonstrations, task weights, device addresses, calibration, and private lab code are retained outside the public repository.
