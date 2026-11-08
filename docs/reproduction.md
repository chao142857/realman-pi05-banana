# Pipeline and interface description

This document describes the workflow used in the experiment. It is not a complete software distribution: the hardware integration relies on an existing lab platform.

## 1. Collect a short, fixed-layout task

Teleoperate the right arm to pick up an artificial banana on the left, place it in the blue plate on the right, release it, and lift clear. Record task success separately from the recorder's file-save status.

The original collection used a USB pedal as a recording input. Robot following was provided by the existing teleoperation system, independently of the recorder.

## 2. Convert and validate

Convert episodes to LeRobot while preserving camera identity, state/action ordering, native units, episode boundaries, and task text. Split episodes before computing normalization statistics.

The experiment checked video decoding, state/action consistency, action-chunk boundaries, TRAIN-only normalization, and normalization round trips before GPU training.

## 3. Preserve the task contract

| Field | Contract |
|---|---|
| State | 14 values: left J1–J6, left gripper, right J1–J6, right gripper |
| Action | Same 14D order; absolute joint target candidates and separate gripper scalars |
| Joint units | Degrees |
| Gripper units | Recorded scalar; retain the independently verified hardware mapping |
| Images | Three RGB views: overhead, left wrist, right wrist |
| Raw image size | 640 × 480; apply the saved processor's resize / pad pipeline |
| Dataset rate | 10 Hz |
| Predicted chunk | 50 actions; five seconds on the dataset timeline |
| Live dispatch | Right arm only; left-arm outputs excluded by the executor |

Instruction used during collection and inference:

> Using the right arm, pick up the artificial banana from its fixed position on the left, place it in the blue plate on the right, release it, and lift the gripper upward clear of the plate.

The model's full 14D interface is retained. The right arm corresponds to indices 7–12, and its gripper to index 13 in zero-based indexing. Apply joint-unit conversions only to the six joints, not to the gripper scalar.

## 4. Fine-tune independent branches

Use the same pretrained base, data split, and saved preprocessing contract for independent Expert-only and LoRA experiments. Verify actual gradients, optimizer updates, complete checkpoint recovery, and held-out VAL metrics. Track sample exposure as well as optimizer steps when comparing different batch sizes.

The public [experiment card](experiment.md) records the selected settings. Follow the official [LeRobot](https://github.com/huggingface/lerobot) and [OpenPI](https://github.com/Physical-Intelligence/openpi) projects for their supported software interfaces. Lab-specific training entry points and patches are not distributed in this initial release.

## 5. Validate offline, then connect observations

Strictly verify the base/update identity, model keys, processor, tokenizer, and TRAIN statistics. Validate finite action chunks and units before connecting live observations.

Next, test camera/state reads with no dispatch. Check camera roles, RGB semantics, observation age, action dimensions, gripper mapping, and inactive-arm isolation. Validate resource use on the actual deployment GPU.

## 6. Evaluate a bounded robot trial

Use the site's established operating procedure, an attended hardware emergency stop, and a checked starting scene. Stopping acknowledgements require follow-up state feedback; a mock or offline pass is not physical safety acceptance.

Record task outcomes independently of program completion. In this pilot the operator stopped or observed trials manually; automatic task-completion detection was not implemented. Freeze the executor and initial-condition protocol before a controlled evaluation.
