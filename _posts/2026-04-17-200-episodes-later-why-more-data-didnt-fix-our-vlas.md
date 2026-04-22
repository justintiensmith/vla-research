---
title: "200 Episodes Later: Why More Data Didn’t Fix Our VLAs"
date: 2026-04-17
author: justintiensmith & mattpidden
excerpt: We collected a larger 200 episode dataset with better visual setup, but still observed the same failures across all models, highlighting the importance of consistent actions over dataset scale.
---

## Context

After our initial experiments with VLAs led to disappointment, we decided to run the setup again with a more controlled dataset. We made three key changes: the object was replaced with one better suited to our hardware, the number of demonstrations was significantly increased, and the world camera was repositioned to reduce occlusions during manipulation.

The previous apple dataset (50 episodes) had already suggested that VLAs struggle with precise manipulation, particularly grasping. However, many successful VLA demos online show strong performance on much more complex tasks, so we wanted to test whether our limitations were simply due to *insufficient data scale*, *task-specific constraints*, or *poor visual setup*.

To investigate this, we collected a larger and more structured dataset using Duplo LEGO blocks.

## The Setup

We replaced the apple with four coloured Duplo blocks (red, green, blue, yellow), which are more stable and consistent in shape, size, and gripability.

We also made a key change to the visual setup:

- the world camera was repositioned to a **front-facing angled view** instead of a strict top-down perspective  
- this reduced occlusions from the robot arm and improved visibility of the workspace  

The goal was to improve both:
- grasp consistency  
- visual observability
- and overall success rate

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

To ensure the results were not an artefact of a specific policy or training setup, we trained multiple models under a range of configurations on the same dataset.

We evaluated three policies:
- SmolVLA  
- ACT  
- pi0.5  

Across these runs, we varied:
- number of training steps  
- which parts of the model were frozen during fine-tuning  
- optimisation and training hyperparameters  

The goal was to test whether performance differences came from the dataset itself, rather than the choice of architecture or training recipe.

## Results

Overall, the results were surprising.

### General performance

Across all models:
- **grasping remained the primary failure mode**
- success rates stayed low (~30–40%)
- the same failure pattern persisted: *miss → retry loop → failure*

Increasing dataset size, removing occlusions from world camera, and changing the object did not resolve the core issue.

### Key observation: pi0.5 generalisation

One notable exception was **pi0.5**, which showed a different behaviour pattern:

- correctly followed language instructions including **colour conditioning**
- successfully selected the correct colour block (even in cases with distractors)
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
  The lack of a single consistent grasping strategy likely limited learning across all models. In our dataset we unintentionally used multiple distinct grasping approaches. For example, in some episodes we rotated the wrist and approached the block in the horizontal plane (see [episode 120](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fjustintiensmith%2Fmulticolour_block_pick_place%2Fepisode_120%3Ft%3D7)). In other cases, we approached vertically from above with the wrist camera offset to the side (see [episode 150](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fjustintiensmith%2Fmulticolour_block_pick_place%2Fepisode_150%3Ft%3D19)). This variation in execution meant the policy was effectively learning multiple incompatible solutions to the same task, preventing it from converging on a precise and reliable grasp strategy.

- **Failure is consistent across architectures**  
  SmolVLA, ACT, and pi0.5 all shared the same grasping bottleneck.

- **pi0.5 showed stronger language grounding**  
  It was the only model that reliably conditioned on colour instructions and generalised beyond training distributions.

- **Grasping is still the dominant bottleneck in the system**  
  Even with improved visual setup and more data, precision manipulation remains unsolved.

## Overall Reflection

Despite collecting a significantly larger dataset, this week felt like a lack of progress.

The models are clearly capable of learning:
- general motion  
- task structure  
- language grounding (to some extent)  

But they consistently fail at the same low-level skill: **precise grasping**.

## Next Steps

Based on these results, the next directions are:

- enforce a **single consistent grasping strategy** during data collection (really important)  
- revisit camera setup (possibly return to top-down but repositioned in front of robot)  
- start some episodes with the robot arm in varying start states in increase recovery performance
- investigate why pi0.5 shows better language grounding than other models  
- focus on improving *action consistency rather than dataset size*

The main takeaway is clear:

> more data did not help, we need consistent action demonstrations
