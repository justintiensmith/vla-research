---
title: "200 Episodes Later: Why More Data Didn’t Fix Our VLAs"
date: 2026-04-17
author: justintiensmith & mattpidden
excerpt: We collected a larger 200-episode dataset with better visual setup, but still observed the same grasping failures across all models — highlighting the importance of consistent actions over dataset scale.
---

## Context

After our initial experiments with VLAs on the SO-101, we wanted to test a simple hypothesis:

> Would scaling up the dataset fix the grasping problem?

The previous apple dataset (50 episodes) suggested that VLAs struggled with precise manipulation, especially grasping. However, most successful VLA demos online show strong performance on significantly harder tasks, so we wanted to investigate whether our limitation was simply *data scale*.

This week, we collected a larger and more structured dataset using Duplo LEGO blocks.

## The Setup

We replaced the apple with four coloured Duplo blocks (red, green, blue, yellow), which are more stable and consistent in shape and gripability.

We also made a key change to the visual setup:

- the world camera was repositioned to a **front-facing angled view** instead of a strict top-down perspective  
- this reduced occlusions from the robot arm and improved visibility of the workspace  

The goal was to improve both:
- grasp consistency  
- visual observability  

## New Dataset: 200 Episodes

We collected a total of **200 teleoperated episodes**:

- 4 block colours (red, green, blue, yellow)  
- 50 episodes per colour  
- task: *pick up the specified coloured block and place it in a black pencil bin*  
- total collection time: ~2 hours  

Each instruction explicitly specified the target colour, e.g.:

> “Pick up the red block and place it in the black bin”

Within each colour group, we also included variation in distractors to increase robustness.

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/datasets/justintiensmith/multicolour_block_pick_place/resolve/main/videos/observation.images.world/chunk-000/file-000.mp4" type="video/mp4">
  </video>  
  <p style="color: #666; font-size: 0.9em; margin: 0 0 0 0;">World camera view (note the angle to avoid any occlusions)</p>
</div>

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/datasets/justintiensmith/multicolour_block_pick_place/resolve/main/videos/observation.images.wrist/chunk-000/file-006.mp4" type="video/mp4">
  </video>  
  <p style="color: #666; font-size: 0.9em; margin: 0 0 0 0;">Wrist camera view</p>
</div>

## Training Experiments

We trained multiple policies on the dataset:

- SmolVLA  
- ACT  
- pi0.5  

We varied:
- training steps  
- freezing configurations  
- optimisation settings  

Despite the larger dataset, training remained stable and loss curves were consistent with earlier runs.

## Results

Overall, the results were surprising.

### General performance

Across all models:
- **grasping remained the primary failure mode**
- success rates stayed low (~30–40%)
- the same failure pattern persisted: *miss → retry loop → failure*

Increasing dataset size alone did not resolve the core issue.

### Key observation: pi0.5 generalisation

One notable exception was **pi0.5**, which showed a different behaviour pattern:

- correctly followed language instructions including **colour conditioning**
- successfully selected the correct block (even in cases with distractors)
- generalised to colour prompts not explicitly seen in fine tuning data  

However:
- it still frequently failed at the grasping stage  

This suggests that:
> semantic task understanding improved, but low-level manipulation did not.

## Takeaways

A few clear conclusions emerged:

- **Scaling the dataset (50 → 200 episodes) did not fix the core issue**  
  More data alone was not sufficient to learn reliable grasping.

- **Consistency of actions is more important than dataset size**  
  The lack of a single consistent grasping strategy likely limited learning across all models.

- **Failure is consistent across architectures**  
  SmolVLA, ACT, and pi0.5 all shared the same grasping bottleneck.

- **pi0.5 showed stronger language grounding**  
  It was the only model that reliably conditioned on colour instructions and generalised beyond training distributions.

- **Grasping is still the dominant bottleneck in the system**  
  Even with improved visual setup and more data, precision manipulation remains unsolved.

## Overall Reflection

Despite collecting a significantly larger dataset, this week felt like a partial step backwards in terms of progress.

The models are clearly capable of learning:
- general motion  
- task structure  
- language grounding (to some extent)  

But they consistently fail at the same low-level skill: **precise grasping**.

## Next Steps

Based on these results, the next directions are:

- enforce a **single consistent grasping strategy** during data collection  
- revisit camera setup (possibly return to top-down but repositioned in front of robot)  
- start some episodes with the robot arm in varying start states in increase recovery performance
- investigate why pi0.5 shows better language grounding than other models  
- focus on improving *action consistency rather than dataset size*

The main takeaway is clear:

> more data did not help — better data structure likely matters more.
