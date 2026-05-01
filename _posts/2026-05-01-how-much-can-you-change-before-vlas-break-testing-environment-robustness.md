---
title: "How Much Can You Change Before VLAs Break? Testing Environment Robustness"
date: 2026-05-01
author: justintiensmith & mattpidden
excerpt: We tested a VLA by changing lighting, camera inputs, and the workspace. The result is strong robustness to visual changes, but surprising fragility to viewpoint and sensor shifts
---

## Context

A common claim around Vision-Language-Action (VLA) models is that they generalise well. 

You’ll often see demos where a robot trained in one setup appears to work in a completely different environment. Different lighting, different backgrounds, slightly different objects, and yet the policy still succeeds.

But “generalisation” is rarely defined precisely. What actually changes between training and deployment? And more importantly, what kinds of changes cause these systems to fail?

This week, we focused on a narrower and more measurable question:

> How robust are VLAs to changes in their environment?


## The Experiment

To isolate environmental generalisation, we fixed everything we could.

All experiments used the same [policy](https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place) (**π0.5**) and the same [dataset](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fjustintiensmith%2Fred_block_precision-multicolour_block_pick_place%2Fepisode_199) that performed best in our previous study. This ensures that any performance differences come from the environment itself, not the model or training process.

We also simplified the task as much as possible. The robot was always asked to:

> “Pick up the red block and place it in the black bin”

To avoid confounding factors, the block was placed in the same 10cmx10cm area for every trial. This means we are not testing spatial generalisation here, only robustness to environmental changes.

We first established a control condition:
- standard lab setup  
- overhead fluorescent lighting  
- wooden worksurface  
- fixed world camera + wrist camera  

This is the condition that the dataset was collected in. The dataset did not contain any variation of the environment such as different lights, worksurfaces etc.

Each experiment was run 10 times, and a trial was considered successful if the task was completed within 60 seconds.

## Environment Variations

We then introduced a series of controlled perturbations, each designed to test a different aspect of robustness.

Some changes were purely visual:
- replacing overhead lighting with a warm lamp  
- adding a grey mat to change the worksurface texture  
- introducing a moving disco light as a visual distractor  

Others affected the robot’s sensing more directly:
- covering the wrist camera entirely  
- shifting the world camera by ~2 feet and rotating it ~30°  

We also tested combinations of changes, such as using both the lamp and the grey mat together, to see how performance degrades under compounding differences.


## Results

| Model       | Normal Conditions | Lamp | Grey Mat | No Wrist Camera | Orange Arm | Mat + Lamp | Paper Cup | Lamp + Disco Light | Shifted World Camera
| :---        | :---              | :--- | :---     | :---            | :---       | :---       |:---       |:---                |:---         
| **Pi0.5**   | 100%              | 100% | 100%     | 40%             | 60%        | 10%        | -%        | 80%                | 10%

The results reveal a clear and somewhat unexpected pattern: **not all environmental changes are equal**.

Some variations had almost no effect at all. Changing the lighting from fluorescent to a warm lamp, or replacing the wooden table with a grey mat, resulted in no drop in performance. Even adding a dynamic visual distractor, like a disco light, only reduced performance slightly to 80%.

This suggests that the model is relatively robust to low-level visual changes such as colour temperature, texture, and background noise. In other words, it is not simply memorising pixel values, it has learned features that transfer across these kinds of variations. This was particularly impressive as the fine tuning dataset only contained a single lighting setup, the same worksurface. We expected because of this the polciy might have overfitted to that envionment.

However, performance drops sharply when the structure of the input changes.

Covering the wrist camera with a lense cap reduces success to 40%, indicating that the policy relies heavily on this viewpoint for precise manipulation. Similarly, changing the robot arm from blue to orange reduces performance to 60%, suggesting some degree of visual overfitting to the robot’s appearance.

The most significant failures occur when spatial perception is disrupted. Shifting and rotating the world camera leads to a collapse in performance (10%), despite the task itself being identical. A similar drop is observed when combining multiple changes (lamp + mat), again resulting in just 10% success.

These results point to an important distinction. The model handles appearance changes (lighting, textures, distractors) quite well, but struggles with geometric and sensor-level changes. When the viewpoint shifts or a key input modality is removed, the policy can no longer reliably map observations to actions.

In effect, the model has learned what to look for, but not fully how that relates to the world in a viewpoint invariant way. It would be interesting to run this experiment on a polciy that was fine tuned on a dataset with diversity in the environment and setup. We would expect the performance to increase.

#### Pi0.5

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/model_c_test_constrained.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 in normal conditions (100%).</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_lamp.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with only a background lamp for lighting (100%).</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_grey_mat.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with a grey mat instead of wooden worksurface (100%).</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_no_wrist_cam.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with the cap blocking the wrist camera (40%).</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_orange.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with an orange SO101 arm (60%).</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_grey_mat_lamp.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with background lamp lighting and a grey mat worksurface (10%).</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_disco.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with a disco light distractor (80%).</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_world_cam_moved.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with a large shift in the world camera poisition and angle (10%).</p>
</div>
