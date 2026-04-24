---
title: "What Makes a Good VLA Dataset? Lessons from Real Experiments"
date: 2026-04-24
author: justintiensmith & mattpidden
excerpt: After repeated failures, we shifted focus from models to data, empirically testing how dataset structure, consistency, and variance affect VLA performance.
---

## Context

After several weeks of disappointing results with VLAs, we decided to change direction.

Instead of continuing to tweak architectures or blindly scale dataset size, we stepped back and asked a more fundamental question:

> Are our failures coming from the models, or from the data?

To answer this properly, we went back to the literature. We revisited papers, blog posts, and real-world VLA implementations to understand what actually drives performance in practice—not just in demos.

## What We Learned from Existing Work

One of the most useful references we found was [ggando’s SO-101 SmolVLA blog](https://ggando.com/blog/smolvla-so101/). Two ideas stood out immediately.

First, **consistency in demonstrations is critical**. Second, **limiting workspace variance is important when learning precise skills like grasping**.

This was a bit of a wake-up call. In our earlier datasets, we had mixed multiple grasping strategies within the same dataset. At the time this felt like “diversity,” but in hindsight it was probably just noise. The model wasn’t learning robustness, it was learning confusion.

Another key takeaway was the idea of **curriculum-style data collection**. Rather than trying to learn everything at once, the suggestion was to:
- start with low-variance, highly consistent demonstrations  
- then gradually introduce variation through further fine-tuning  

This framing ended up heavily influencing how we structured our experiments.

A second reference that really resonated was the [LeRobot team’s robot folding work](https://huggingface.co/spaces/lerobot/robot-folding). They made a point that feels increasingly obvious once you notice it:

> “Flashy demos of robotic systems are popping up… but we typically don’t know how these systems were actually built and trained.”

Looking into their setup reinforced that point. Their final results came from a long and messy data process:

- **5,688 episodes (131 hours)** of teleoperation were initially collected, much of which turned out to be low quality  
- they then curated a smaller dataset of **~1,200 high-quality episodes (30 hours)** for training  
- performance only improved after enforcing **consistent action strategies across operators**

Even then, achieving strong performance required a significant amount of carefully controlled data.

This mirrored our own experience almost exactly. Simply adding more data wasn’t helping. What mattered far more was how that data was collected.

We identified several issues in our own setup:
- inconsistent grasping strategies across demonstrations  
- suboptimal camera placement (our wrist camera was mounted on the side rather than above)  
- no clear structure or philosophy behind dataset design  

At that point, it became clear that if we wanted to make progress, we needed to isolate and test the effect of dataset design itself.

## Experimental Setup

To do this, we designed a controlled experiment focused entirely on **dataset structure**.

We fixed the model to **π0.5**, which had shown the most promising behaviour so far, and removed as many other variables as possible. That meant keeping the hardware, lighting, camera setup, and control strategy identical across all experiments.

We then collected two new datasets, each with **100 episodes**, but with very different characteristics.

#### [Dataset 1](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fjustintiensmith%2Fmulticolour_block_pick_place_2%2Fepisode_0) — Low Variance (Precision-Focused)

This dataset was intentionally simple and tightly controlled. It was designed to answer one question: *can the model learn a precise, repeatable skill under ideal conditions?*

- single object (red block)  
- fixed target bin  
- block constrained to a **10cm × 10cm region**  
- highly consistent demonstrations  

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/datasets/justintiensmith/multicolour_block_pick_place_2/resolve/main/videos/observation.images.world/chunk-000/file-000.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">World camera view (dataset 1)</p>
</div>

#### [Dataset 2](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fmattpidden%2Fconsistent-block-pick-and-place-into-basket%2Fepisode_0) — High Variance (Generalisation-Focused)

In contrast, the second dataset was designed to introduce variability and test generalisation.

- 4 block colours (25 episodes each)  
- varying object positions  
- moving target bin (within roughly a **20cm × 20cm region**)  
- increased task diversity  

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/datasets/mattpidden/precise_multicolour_block_pick_place/resolve/main/videos/observation.images.world/chunk-000/file-006.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">World camera view (dataset 2)</p>
</div>

## Training

We then trained π0.5 under four different conditions to understand how these datasets interact:

- **A)** trained on Dataset 1 only  
- **B)** trained on Dataset 2 only  
- **C)** trained on the union of both datasets  
- **D)** trained on Dataset 1, then fine-tuned on the union  

This setup lets us probe a few important questions:
- Is precision or diversity more important?  
- Does mixing datasets help or hurt?  
- Does a curriculum (structured progression) outperform naive data combination?  

Training was kept consistent across runs. For reference, here’s the configuration we used:

```bash
lerobot-train \
  --dataset.repo_id=author/repoid \
  --policy.type=pi05 \
  --output_dir=/vol/outputdir \
  --job_name=pi05_training \
  --policy.pretrained_path=lerobot/pi05_base \
  --policy.dtype=bfloat16 \
  --policy.gradient_checkpointing=true \
  --policy.freeze_vision_encoder=false \
  --policy.train_expert_only=false \
  --steps=5000 \
  --batch_size=32 \
  --save_freq 1000 \
  --policy.device=cuda \
  --policy.repo_id=author/outputrepo \
  --policy.compile_model=false \
  --policy.normalization_mapping='{"ACTION": "MEAN_STD", "STATE": "MEAN_STD", "VISUAL": "IDENTITY"}'
  ```

## Results 

We evaluated each model across four progressively more challenging test scenarios, running 10 trials per test to ensure consistency.

The first test measured performance under tightly controlled conditions. The red block was constrained to the same 10cm × 10cm region used in Dataset 1, allowing us to isolate how well each model learned precise, low-variance behaviour.

We then increased the difficulty by expanding the workspace. In this second test, the red block could appear anywhere within the robot’s reachable area, introducing significantly more spatial variation.

Next, we evaluated robustness to distractions. The task was changed to picking up the green block (seen during training in Dataset 2), while introducing additional blocks of different colours as distractors. This tested whether the model could correctly follow instructions rather than relying on simple heuristics.

Finally, we ran an out-of-distribution test. The model was asked to pick up a purple block, which it had never seen during fine-tuning, again in the presence of other coloured distractors. This provides a measure of true generalisation beyond the training distribution.

A summary of the results across all models and test conditions is shown in the table below.

| Model   | Constrained       | Spatial           | Distractor        | OOD               | AVG               |
|:--------|:------------------|:------------------|:------------------|:------------------|:------------------|
| Model A | 90% (23.8s)       | 0% (n/a)          | 0% (n/a)          | 0% (n/a)          | 23% (23.8s)       |
| Model B | 100% (24.2s)      | 70% (30.4s)       | 80% (27.8s)       | 70% (24.8s)       | 80% (26.8s)       |
| Model C | 100% (24.2s)      | 70% (30.4s)       | 80% (27.8s)       | 70% (24.8s)       | 80% (26.8s)       |
| Model D | 90% (32.1s)       | 50% (28.4s)       | 25% (27.3s)       | 50% (29.5s)       | 54% (29.3s)       |


#### [Model A](https://huggingface.co/mattpidden/pi05_5k_multicolour_block_pick_place_2)


<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_multicolour_block_pick_place_2/resolve/main/model_a_test_constrained.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model A in constrained conditions with prompt "Pick up the red block and carefully place it in the black bin" running fully autonomously.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_multicolour_block_pick_place_2/resolve/main/model_a_test_spatial.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model A with spatial diversity and prompt "Pick up the red block and carefully place it in the black bin" running fully autonomously.</p>
</div>


#### [Model B](https://huggingface.co/mattpidden/pi05_5k_diverse_multicolour_block_pick_place)

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_diverse_multicolour_block_pick_place/resolve/main/model_b_test_constrained.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model B in constrained conditions with prompt "Pick up the red block and carefully place it in the black bin" running fully autonomously.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_diverse_multicolour_block_pick_place/resolve/main/model_b_test_spatial.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model B with spatial diversity and prompt "Pick up the red block and carefully place it in the black bin" running fully autonomously.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_diverse_multicolour_block_pick_place/resolve/main/model_b_test_distractor.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model B with distractors and prompt "Pick up the green block and carefully place it in the black bin" running fully autonomously.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_diverse_multicolour_block_pick_place/resolve/main/model_b_test_ood.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model B with distractors and prompt "Pick up the purple block and carefully place it in the black bin" running fully autonomously</p>
</div>


#### [Model C](https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place)

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/model_c_test_constrained.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model C in constrained conditions with prompt "Pick up the red block and carefully place it in the black bin" running fully autonomously.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/model_c_test_spatial.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model C with spatial diversity and prompt "Pick up the red block and carefully place it in the black bin" running fully autonomously.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/model_c_test_distractor.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model C with distractors and prompt "Pick up the green block and carefully place it in the black bin" running fully autonomously.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/model_c_test_ood.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model C with distractors and prompt "Pick up the purple block and carefully place it in the black bin" running fully autonomously</p>
</div>


#### [Model D](https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place_stage2)

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place_stage2/resolve/main/model_d_test_constrained.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model D in constrained conditions with prompt "Pick up the red block and carefully place it in the black bin" running fully autonomously.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place_stage2/resolve/main/model_d_test_spatial.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model D with spatial diversity and prompt "Pick up the red block and carefully place it in the black bin" running fully autonomously.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place_stage2/resolve/main/model_d_test_distractor.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model D with distractors and prompt "Pick up the green block and carefully place it in the black bin" running fully autonomously.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place_stage2/resolve/main/model_d_test_ood.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of model D with distractors and prompt "Pick up the purple block and carefully place it in the black bin" running fully autonomously</p>
</div>

