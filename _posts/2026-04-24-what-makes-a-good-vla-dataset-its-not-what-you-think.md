---
title: "What Makes a Good VLA Dataset? (It’s Not What You Think)"
date: 2026-04-22
author: justintiensmith & mattpidden
excerpt: After repeated failures, we shifted focus from models to data, empirically testing how dataset structure, consistency, and variance affect VLA performance.
---

## Context

After several weeks of disappointing results with VLAs, we took a step back.

Rather than continuing to tweak models or scale datasets blindly, we wanted to understand a more fundamental question:

> Are our failures due to the models or the data?

To answer this, we spent time reviewing papers, blog posts, and real world implementations of VLAs to better understand what we might be missing.

## What We Learned from Existing Work

One particularly useful reference was [ggando’s SO-101 SmolVLA blog](https://ggando.com/blog/smolvla-so101/), which highlighted two key ideas:

- **consistent demonstrations are critical**  
- **limiting workspace variance helps learn precision tasks like grasping**

This immediately stood out. In our previous datasets, we had used multiple grasping strategies within the same dataset, which likely made learning unstable.

Another insight was around **curriculum-style data collection**:
- start with low-variance, highly consistent demonstrations  
- then gradually introduce variation through further fine-tuning  

A second important reference was the LeRobot team’s work on robot folding:

> “Flashy demos of robotic systems are popping up… but we typically don’t know how these systems were actually built and trained.”

We strongly agree with this.

What stood out most was the scale and process behind their results:

- **5,688 episodes (131 hours)** of teleoperation collected which turned out to be mostly poor quality demos
- a new dataset of **~1,200 high-quality episodes (30 hours)** for final training  
- performance only improved after enforcing **consistent action strategies across operators**

Even then, large amounts of data were required to achieve high success rates.

This directly mirrors our experience:
- more data alone did not improve performance  
- **data quality and consistency mattered significantly more**

We identified issues in our own system:

- inconsistent grasping strategies across demonstrations  
- suboptimal camera placement (wrist camera mounted on the side rather than top)  
- lack of a structured approach to dataset design  



## Experimental Setup

To isolate the effect of **dataset structure**, we designed a controlled experiment using a single policy: **pi0.5** (which had shown the most promising behaviour so far).

We collected two new datasets with **100 episodes each**, keeping:
- identical hardware  
- identical grasping strategy  
- consistent lighting conditions  
- same camera setup (wrist + world camera)

### [Dataset 1](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fjustintiensmith%2Fmulticolour_block_pick_place_2%2Fepisode_0): Low Variance (Precision-Focused)

- single object: red block  
- fixed target bin  
- block constrained to a **10cm × 10cm region**  
- highly consistent demonstrations  

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/datasets/justintiensmith/multicolour_block_pick_place_2/resolve/main/videos/observation.images.world/chunk-000/file-000.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">World camera view (dataset 1)</p>
</div>


### [Dataset 2](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fmattpidden%2Fprecise_multicolour_block_pick_place%2Fepisode_23): High Variance (Generalisation-Focused)

- 4 block colours (25 episodes each)  
- varying object positions  
- moving target bin (approx in 20cm × 20cm region)  
- increased task diversity  

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/datasets/mattpidden/precise_multicolour_block_pick_place/resolve/main/videos/observation.images.world/chunk-000/file-006.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">World camera view (dataset 2)</p>
</div>



## Training

We trained pi0.5 under four different conditions:

- **A)** trained on Dataset 1 only  
- **B)** trained on Dataset 2 only  
- **C)** trained on the union of both datasets  
- **D)** trained on Dataset 1 → then fine-tuned on union of Dataset 1 & 2

This allows us to test:
- precision vs generalisation  
- data mixing vs staged training  
- effects of curriculum-style learning  

Training Parameters for Pi0.5 (TODO)



## Results 

### Model A

Constrained test (10cmx10cm grid red cube)
Success rate: 90%
Average Time to complete: 22,25,26,23,22,24,22,28,22

Spatial test (Anywhere red cube)
Success rate: 0%
Average Time to complete: infinity

Distractor robustness test (anywhere green cube + 2 other random colours)
Success rate: 0%
Average Time to complete: 

OOD test (purple cube anywhere + 2 other random colours)
Success rate: 0%
Average Time to complete: 

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_multicolour_block_pick_place_2/resolve/main/model_a_indist.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Example of in-distribution task success (model A)</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_multicolour_block_pick_place_2/resolve/main/model_a_outdist.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Example of out-of-distribution task failure (model A)</p>
</div>

The model learned:
- precise grasping within the constrained region  
- reliable execution of the full task  

However, it completely failed to generalise beyond that region.

- replacing the block with a different colour → still grasped successfully  
- introducing multiple blocks → model often selected randomly  
- if the wrong block was picked first:
  - the model recovered and completed the correct task of picking the red block

This suggests:
- strong spatial overfitting  
- partial task understanding  
- limited but non-trivial recovery behaviour

### Model B


### Model C
Constrained test (10cmx10cm grid red cube)
Success rate: 100%
Average Time to complete: 24.2s

Spatial test (Anywhere red cube)
Success rate: 70%
Average Time to complete: 30.4s

Distractor robustness test (anywhere green cube + 2 other random colours)
Success rate: 80%
Average Time to complete: 27.8s

OOD test (purple cube anywhere + 2 other random colours)
Success rate: 7/10
Average Time to complete: 24.8s

### Model D
Constrained test (10cmx10cm grid red cube)
Success rate: 90%
Average Time to complete: 32.1s

Spatial test (Anywhere red cube)
Success rate: 50%
Average Time to complete: 28.4s

Distractor robustness test (anywhere green cube + 2 other random colours)
Success rate: 25%
Average Time to complete: 27.3s

OOD test (purple cube anywhere + 2 other random colours)
Success rate: 50%
Average Time to complete: 29.5s
