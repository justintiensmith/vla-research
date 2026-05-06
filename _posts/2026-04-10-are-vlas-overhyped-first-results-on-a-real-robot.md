---
title: "Are VLAs Overhyped? First Results on a Real Robot"
date: 2026-04-10
author: mattpidden
excerpt: Early experiments with VLAs on a real robot suggest that few-shot learning capabilities are overstated, with performance heavily dependent on data quality.
---

## Context

If you watch robotics demos online, Vision-Language Action (VLA) models look like magic. These models are advertised as being able to generalise after a few demonstrations. However, based on preliminary experiments, I found that getting an SO-101 arm to reliably pick and place an apple into a bowl was not as simple as I thought.

This week I focused on setting up a full end-to-end pipeline: setting up the robot → collecting a dataset → finetuning a VLA → doing real-world inference.

<div style="border-left: 4px solid #4a90e2; background: #f7f9fc; padding: 12px 16px; margin: 20px 0; border-radius: 6px;">
  <strong>What is a Vision-Language Action (VLA) model?</strong>
  <p style="margin-top: 8px;">
    A VLA is an artificial intelligence (AI) model that takes in images (vision) and a natural language instruction (i.e., place the apple in the bowl), and outputs actions for a robot to execute. 
    Instead of explicitly programming the behaviour, the model learns from demonstrations how to map 
    <em>(what it sees + what it’s told)</em> → <em>what action to take next</em>.
  </p>
  <p style="margin-top: 8px;">
    What makes VLAs particularly interesting is that they build on large pretrained vision and language models, allowing them to generalise across tasks and instructions in ways that traditional imitation learning, reinforcement learning, or classical planning-based robotics systems typically cannot without significant task-specific engineering.
  </p>
  <p style="margin-top: 8px;">
    To use VLAs in practice, users have to collect teleoperated examples of a task (e.g., pick-and-place) for their specific hardware setup (robot + camera positions), then fine-tune a pretrained model 
    so it can generalise that behaviour to new situations it's never seen before.
  </p>
</div>

## The Setup

I set up the open source [SO-101 robot arm](https://huggingface.co/docs/lerobot/en/so101) and followed the LeRobot installation process to create the Python environment. The installation itself was mostly smooth, although getting everything working cleanly on Imperial College's GPU cluster took some effort later on.

After setup, I calibrated the arms (joint limits, motor ranges) using the `lerobot-calibrate` command line interface (CLI) command and spent time practising teleoperation using the leader arm to get comfortable controlling the follower.

## The Task

The task was to pick up an apple and place it into a white ceramic bowl. The language instruction I provided the VLAs with was:

> Put the apple in the bowl

Each episode consisted of a left-to-right pick-and-place motion, with some variation in object and target positions.

## Dataset v1: 20 Episodes

The initial dataset consisted of **20 teleoperated demonstrations**, collected using `lerobot-record`. The cameras I included are as follows:

- a wrist-mounted camera on the follower arm  
- a world camera mounted above the workspace (top-down view)

I initially started with just 20 demonstrations as I was under the impression that would suffice for a VLA to learn a task. You can view the full dataset [here](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fmattpidden%2Fsmol-vla-test-dataset%2Fepisode_0).

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/datasets/mattpidden/smol-vla-test-dataset/resolve/main/videos/apple-dataset-timelapse.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0 0 0 0;">Timelapse of data collection process</p>
</div>

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/datasets/mattpidden/smol-vla-test-dataset/resolve/main/videos/observation.images.world/chunk-000/file-000.mp4" type="video/mp4">
  </video>  
  <p style="color: #666; font-size: 0.9em; margin: 0 0 0 0;">World camera view (note the occlusions of apple)</p>
</div>

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/datasets/mattpidden/smol-vla-test-dataset/resolve/main/videos/observation.images.claw/chunk-000/file-001.mp4" type="video/mp4">
  </video>  <p style="color: #666; font-size: 0.9em; margin: 0 0 0 0;">Claw camera view</p>
</div>

The demonstrations were unstructured, with moderate variation in object and target starting positions.

## Training

I fine-tuned **SmolVLA** using LeRobot.

Training was done on Imperial’s GPU cluster using Slurm. This introduced some friction, including dependency issues during setup and occasional out-of-memory errors.

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

Performance on the real robot, however, was poor:

- the robot consistently failed to grasp the apple  
- often missed the object entirely  
- when manually corrected into a grasp, it sometimes moved toward the bowl but dropped the object outside the bowl

**Success rate: ~0%**

That said, the behaviour wasn’t random. The model appeared to partially understand the task structure of moving toward the bowl after interacting with the object, but it could not execute a reliable grasp.

Grasping was clearly the main failure point.

## Dataset v2: 50 Episodes

After some further reading of other blog posts, I read that to improve performance, I needed more and better data. I collected **30 additional episodes** (50 total), with a key change:

- reduced task variability  
- constrained apple and bowl positions to tighter regions  
- more consistent demonstration trajectories  

The goal was to increase demonstration density rather than diversity.

## Results (v2)

After fine-tuning on the expanded dataset, I got a **~30% success rate (3/10 trials)**. I classified a run as successful if the apple ended in the bowl within 60 seconds.

This was a clear improvement over v1. The model began to complete the full task in some cases, although behaviour was still inconsistent.

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/smolvla_apple_policy2/resolve/main/smolvla-apple-succeed.mp4" type="video/mp4">
  </video>
    <p style="color: #666; font-size: 0.9em; margin: 0 0 0 0;">Example of successful task completion (v2, SmolVLA, 50 demos, 40k fine tuning steps)</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/smolvla_apple_policy2/resolve/main/smolvla-apple-fail.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0 0 0 0;">Example of failed task (v2, SmolVLA, 50 demos, 40k fine tuning steps)</p>
</div>

## What Changed?

The main difference wasn’t just more data — it was *more consistent data*.

By reducing spatial variability, the model was better able to learn precise grasp locations rather than a broad, under-specified policy.

## Additional Policies: pi0.5 and ACT

To better understand whether the limitations were specific to SmolVLA or more general, I trained additional policies on the same 50-episode apple dataset.

### pi0.5

I initially attempted to fine-tune pi0 and pi0-fast, but ran into issues getting them to train correctly, so I moved to pi0.5, which is the newest of Physical Intelligence's open source models and is supposed to generalise better than pi0 and pi0-fast anyways.

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

In practice, this means the first grasp attempt largely determines the outcome of the entire episode

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
  20 demonstrations were not sufficient to learn this task. Even at 50 demonstrations, performance remained inconsistent. VLAs did not exhibit strong few-shot behaviour in this real-world setting. I was genuinely surprised at the poor performance. Online video demonstrations led me to believe VLAs could achieve high success rates with minimal fine-tuning for simple tasks like pick-and-place and generalise easily.

- **Grasping is the dominant bottleneck**  
  Across each of the policies (SmolVLA, pi0.5, ACT) tested, failure was almost always due to inaccurate grasping. Once the object was successfully picked up, task completion was likely.

- **The first action largely determines the outcome**  
  Policies rarely recover from a failed grasp. Early errors propagate through the rest of the trajectory. A solution to this would be to include different starting points in the training episodes.

- **Policy choice affects behaviour, but not failure mode**  
  Different models produced noticeably different motion styles (e.g. smooth vs jittery), but all suffered from the same underlying grasping issue.
