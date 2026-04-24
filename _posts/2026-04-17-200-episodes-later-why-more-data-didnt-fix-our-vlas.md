---
title: "200 Episodes Later: Why More Data Didn’t Fix Our VLAs"
date: 2026-04-17
author: justintiensmith & mattpidden
excerpt: We collected a larger 200 episode dataset with better visual setup, but still observed the same failures across all models, highlighting the importance of consistent actions over dataset scale.
---

## Context

After our initial experiments with VLAs led to disappointing results, we decided to rerun the setup with a more controlled dataset. This time, we made three deliberate changes: we replaced the object with one better suited to our hardware, increased the number of demonstrations significantly, and adjusted the world camera to reduce occlusions during manipulation.

The earlier apple dataset (50 episodes) had already hinted that VLAs struggle with precise manipulation, especially grasping. However, since many public demos show strong performance on far more complex tasks, we wanted to test whether our limitations were simply due to insufficient data, poor visual setup, or task specific constraints.

To investigate this properly, we moved to Duplo LEGO blocks and rebuilt the dataset from scratch.

## The Setup

We replaced the apple with four coloured Duplo blocks (red, green, blue, yellow). These were chosen because they are more stable in shape, size, and grip consistency, which should in theory make the manipulation problem easier.

We also changed the camera configuration. Instead of a strict top-down view, the world camera was moved to a front-facing angled position. This reduced occlusions from the robot arm and improved visibility of the workspace during interaction.

Overall, the goal was simple: improve grasp consistency, improve visual observability, and see whether either of those changes would translate into better success rates.

## New Dataset: 200 Episodes

We collected a total of 200 teleoperated episodes structured evenly across the four block colours:

- 4 colours (red, green, blue, yellow)
- 50 episodes per colour
- task: pick up the specified coloured block and place it in a black pencil bin
- total collection time: approximately 2 hours

Each episode was paired with a language instruction such as:

> “Pick up the red block and place it in the black bin”

Within each colour group, we also included variation in distractors to increase robustness.

Two views were recorded throughout:
- a world camera with the new angled setup
- a wrist mounted camera for fine grained manipulation detail

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/datasets/justintiensmith/multicolour_block_pick_place/resolve/main/videos/observation.images.world/chunk-000/file-000.mp4" type="video/mp4">
  </video>  
  <p style="color: #666; font-size: 0.9em; margin: 0 0 0 0;">World camera view (note how the angle avoids occlusions)</p>
</div>

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/datasets/justintiensmith/multicolour_block_pick_place/resolve/main/videos/observation.images.wrist/chunk-000/file-006.mp4" type="video/mp4">
  </video>  
  <p style="color: #666; font-size: 0.9em; margin: 0 0 0 0;">Wrist camera view</p>
</div>

## Training Experiments

To ensure results were not tied to a single implementation choice, we trained multiple models on the same dataset under different configurations.

We evaluated three policies: SmolVLA, ACT, and pi0.5. Across these runs, we varied training steps, freezing strategies, and optimisation settings.

The goal here was to isolate whether performance differences came from the dataset itself or from model architecture and training recipe.

## Results

Across all models, the same pattern emerged. Grasping remained the dominant failure mode, and overall success rates stayed low at roughly 30–40%. Increasing dataset size, improving camera placement, and switching to a more stable object did not meaningfully change this outcome.

#### Key observation: pi0.5 generalisation

One interesting exception was pi0.5. Unlike the other models, it showed noticeably stronger language grounding. It correctly followed colour conditioned instructions and was able to select the correct block even in the presence of distractors.

In some cases, it also generalised to colour prompts that were not explicitly emphasised during finetuning.

However, this improvement was limited to task understanding. It still failed frequently at the physical grasping stage, suggesting that while semantic grounding improved, low level control did not.

## Takeaways

First, increasing dataset size from 50 to 200 episodes did not resolve the core issue. More data alone was not enough to produce reliable grasping behaviour.

Second, consistency in action strategy appears to matter far more than raw dataset scale. In hindsight, our demonstrations contained multiple inconsistent grasping styles. Some episodes involved horizontal approaches with wrist rotation (see [episode 120](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fjustintiensmith%2Fmulticolour_block_pick_place%2Fepisode_120%3Ft%3D7)), while others used a vertical top-down grasp with different camera relative positioning (see [episode 150](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fjustintiensmith%2Fmulticolour_block_pick_place%2Fepisode_150%3Ft%3D19)). This meant the model was effectively learning multiple conflicting solutions to the same task rather than converging on a single stable strategy.

Third, the failure mode was shared across architectures. SmolVLA, ACT, and pi0.5 all struggled with the same underlying grasping bottleneck.

Fourth, pi0.5 stood out for its stronger language grounding. It was the only model that consistently conditioned on colour instructions and generalised beyond strict training distribution constraints.

Finally, grasping remains the limiting factor in the system. Even with improved visuals and more data, precise manipulation is still the dominant failure point.

## Overall Reflection

Despite collecting a significantly larger dataset and improving the experimental setup, progress felt limited this week.

The models clearly learn certain aspects of the task well, including motion primitives, high level structure, and to some extent language grounding. However, they consistently fail at the same low level skill: precise grasping.

## Next Steps

Based on these results, the next direction is becoming clearer.

We need to enforce a single consistent grasping strategy during data collection, as this appears to be a key factor limiting performance. Camera setup should also be revisited, potentially returning to a top-down configuration while improving placement to reduce occlusion issues.

We also want to explore varying initial arm states to improve robustness and recovery behaviour, and investigate why pi0.5 appears to have stronger language grounding than the other models.

Most importantly, the focus shifts away from scaling dataset size and towards improving action consistency.

The main takeaway is simple: more data did not help. Consistency did.
