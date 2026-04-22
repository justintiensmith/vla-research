---
title: "Are VLAs Overhyped? First Results on a Real Robot"
date: 2026-04-10
author: mattpidden
excerpt: Early experiments with VLAs on a real robot suggest few-shot learning is overstated, with performance heavily dependent on data quality.
---

## Context

This week was about getting a full end-to-end pipeline working: robot setup → data collection → training → real-world inference.

The goal was simple: get a Vision-Language Action (VLA) model running on the SO-101 and see how quickly we could get *any* level of real-world performance on a basic pick-and-place task.

## The Setup

I set up the SO-101 robot arm and followed the LeRobot installation process to create the Python environment. The installation itself was mostly smooth, although getting everything working cleanly on the GPU cluster took some effort later on.

After setup, I calibrated the arms (joint limits, motor ranges) using `lerobot-calibrate` CLI command and spent time teleoperating using the leader arm to get comfortable controlling the follower.

## The Task

The task was to pick up an apple and place it into a white ceramic bowl. The language instruction I provided the VLAs was

> Put the apple in the bowl

Each episode consisted of a left-to-right pick-and-place motion, with some variation in object and target positions.

## Dataset v1: 20 Episodes

The initial dataset consisted of **20 teleoperated demonstrations**, collected using `lerobot-record` and:

- a wrist-mounted camera on the follower arm  
- a world camera mounted above the workspace (top-down view)

I initally started with just 20 demonstrations as I was under the impression that would suffice for a VLA to learn a task. You can view the full dataset [here](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fmattpidden%2Fsmol-vla-test-dataset%2Fepisode_0).

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  Timelapse of data collection process<br>
  <video controls width="100%">
    <source src="https://huggingface.co/datasets/mattpidden/smol-vla-test-dataset/resolve/main/videos/apple-dataset-timelapse.mp4" type="video/mp4">
  </video>
</div>

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  View of world camera during data collection <br>
  <video controls width="100%">
    <source src="[https://huggingface.co/datasets/mattpidden/smol-vla-test-dataset/resolve/main/videos/apple-dataset-timelapse.mp4](https://huggingface.co/datasets/mattpidden/smol-vla-test-dataset/resolve/main/videos/observation.images.world/chunk-000/file-000.mp4)" type="video/mp4">
  </video>
</div>

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  View of claw camera data collection <br>
  <video controls width="100%">
    <source src="[https://huggingface.co/datasets/mattpidden/smol-vla-test-dataset/resolve/main/videos/apple-dataset-timelapse.mp4](https://huggingface.co/datasets/mattpidden/smol-vla-test-dataset/resolve/main/videos/observation.images.claw/chunk-000/file-001.mp4)" type="video/mp4">
  </video>
</div>

The demonstrations were unstructured, with moderate variation in object and target starting positions.

## Training

I fine-tuned **SmolVLA** using LeRobot.

Training was done on Imperial’s GPU cluster using Slurm. This introduced some friction:
- dependency issues during setup  
- occasional out-of-memory errors  

Eventually, I got training running on an A40 node via the LeRobot CLI with the following command and flags.
```bash
lerobot-train \
  --dataset.repo_id=mattpidden/smol-vla-test-dataset \
  --policy.path=lerobot/smolvla_base \
  --output_dir=/vol/bitbucket/mdp25/outputs/smolvla2 \
  --job_name=smolvla_training \
  --batch_size=64 \
  --steps 40000 \
  --save_freq 1000 \
  --policy.device=cuda \
  --policy.repo_id=mattpidden/smolvla_apple_policy \
  --rename_map='{"observation.images.claw": "observation.images.camera1", "observation.images.world": "observation.images.camera2"}'
```

## Results (v1)
For inference, I used a local RTX 4090 with a LeRobot async server and a chunk size of 50 with threshold of 0.5. Runtime performance was good (~0.1s per chunk), so latency was not a bottleneck.

Performance on the real robot was poor:

- the robot consistently failed to grasp the apple  
- often missed the object entirely  
- when manually corrected into a grasp, it sometimes moved toward the bowl but dropped the object nearby

**Success rate: ~0%**

That said, the behaviour wasn’t random. The model appeared to partially understand the task structure — moving toward the bowl after interacting with the object — but could not execute a reliable grasp.

Grasping was clearly the main failure point.

## Dataset v2: 50 Episodes

After some further reading of other blogs posts I read that to improve performance I needed more and better data. I collected **30 additional episodes** (50 total), with a key change:

- reduced task variability  
- constrained apple and bowl positions to tighter regions  
- more consistent demonstration trajectories  

The goal was to increase demonstration density rather than diversity.

## Results (v2)

After fine-tuning on the expanded dataset:

- **~30% success rate (3/10 trials)**  
- a run was classified as successful if the apple ended in the bowl within 60 seconds  

This was a clear improvement over v1. The model began to complete the full task in some cases, although behaviour was still inconsistent.

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  Example of successful task completion (v2, SmolVLA, 50 demos, 40k fine tuning steps)<br>
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/smolvla_apple_policy2/resolve/main/smolvla-apple-succeed.mp4" type="video/mp4">
  </video>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  Example of failed task (v2, SmolVLA, 50 demos, 40k fine tuning steps)<br>
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/smolvla_apple_policy2/resolve/main/smolvla-apple-fail.mp4" type="video/mp4">
  </video>
</div>

## What Changed?

The main difference wasn’t just more data — it was *more consistent data*.

By reducing spatial variability, the model was better able to learn precise grasp locations rather than a broad, under-specified policy.

## Additional Policies: pi0.5 and ACT

To better understand whether the limitations were specific to SmolVLA or more general, I trained additional policies on the same 50-episode apple dataset.

### pi0.5

I first trained **pi0.5**. I initially attempted to fine-tune pi0 and pi0-fast, but ran into issues getting them to train correctly, so moved to pi0.5, which is also expected to generalise better.

Training was done with the following key flags:
```bash
--freeze_vision_encoder=false
--train_expert_only=true
--steps=40000
```

In hindsight, these settings were likely suboptimal, as they may have trained the wrong parts of the model.

Inference ran at ~0.2s per step on the RTX 4090. Overall performance was similar to SmolVLA, with a success rate of approximately **~30%**.

However, the behaviour was noticeably different:

- pi0.5 moved **more confidently and decisively**  
- SmolVLA appeared **slower, more jittery, and hesitant**  

Despite this difference in motion, both models failed in similar ways. The policy would repeatedly attempt to grasp slightly offset from the apple and miss.

### ACT

I also trained an **ACT policy** for 10k steps on the same dataset.

Compared to both SmolVLA and pi0.5:

- motion was **smoother and more stable**  
- performance was slightly higher at **~40% success**  
- execution appeared more consistent overall  

## Shared Failure Mode

Across all three models, the same core issue emerged:

- **grasping is the primary bottleneck**  
- models frequently miss the apple by a small margin  
- once a grasp fails, they enter a loop:  
  - re-approach → miss → jitter → repeat  
- recovery after a failed grasp is rare  

If the object was successfully grasped:

- placing into the bowl was usually successful  
- overall task success became highly likely  

In practice, this means:

> the first grasp attempt largely determines the outcome of the entire episode

## Hypotheses

There are several possible reasons for the poor grasping performance:

- the apple is relatively large, requiring the gripper to fully open and align with precision 
- the surface is reflective, introducing visual ambiguity  
- the object is often ocluded by the robot arm in the top-down camera view, limiting its ability to provide useful information

A more favourable setup might include:

- a smaller object  
- a less reflective surface  
- an angled camera to provide better depth cues  

## Takeaways

A few key insights emerged from these experiments:

- **Few-shot performance is overstated in practice**  
  20 demonstrations were not sufficient to learn this task. Even at 50 demonstrations, performance remained inconsistent. VLAs did not exhibit strong few-shot behaviour in this real-world setting. I was genuinely surprised at the poor performance. Online video demonstrations led me to belive VLAs could achieve high success rates with minimal fine tuning for simple task and generalise easily.

- **Grasping is the dominant bottleneck**  
  Across all policies (SmolVLA, pi0.5, ACT), failure was almost always due to inaccurate grasping. Once the object was successfully picked up, task completion was likely.

- **The first action largely determines the outcome**  
  Policies rarely recover from a failed grasp. Early errors propagate through the rest of the trajectory. A solution to this would be to include different starting points in the training episodes.

- **Policy choice affects behaviour, but not failure mode**  
  Different models produced noticeably different motion styles (e.g. smooth vs jittery), but all suffered from the same underlying grasping issue.
