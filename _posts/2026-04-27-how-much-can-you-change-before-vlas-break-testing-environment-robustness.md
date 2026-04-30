---
title: "How Much Can You Change Before VLAs Break? Testing Environment Robustness"
date: 2026-04-27
author: justintiensmith & mattpidden
excerpt: We stress-tested our VLA setup by changing the lighting, worksurface, and camera configuration. The results? VLAs are ______.
---

## The Generalisation Claim

One of the primary selling points of VLAs is their ability to generalise across diverse environments. But "generalisation" is often a vague term used in papers; this week, we decided to experiment with VLA generalisation to diverse environments.

## The Experiment

All of the models we selected were trained on the same dataset to ensure a fair comparison. We used the dataset that provided the best performance across various model architectures in our previous experiment. You can find the dataset [here](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fjustintiensmith%2Fred_block_precision-multicolour_block_pick_place%2Fepisode_199).

The dataset consists of 200 episodes:
* **Precision Skill (100 episodes):** Picking up a red block within a defined 10cm x 10cm square area.
* **Diversity Skill (100 episodes):** Picking up blocks of different colors in a range of locations around the workspace.

For our experiments, we established a **Control Experiment** where we ran inference 10 times in the same setup as the data collection: a standard workspace table, fluorescent lab lighting, and a fixed world camera position. 

We defined success as completing the task within 60 seconds. To keep the testing focused, we only placed the block within the 10cm x 10cm "precision" square and used only the red block for all inference runs.

We moved the world camera roughly 2 feet away and then rotated the camera by roughly 30 degree. IT consistantly was able to pick up the block but couldn't place it in the bin; only missed the pick up once. It always dropped it on the right side of the bin and it was close to going in,hitting the rim. It was able to grasp it every time except once.

## Results

| Model       | Normal Conditions | Lamp | Grey Mat | No Wrist Camera | Orange Arm | Mat + Lamp | Paper Cup | Lamp + Disco Light | Shifted World Camera
| :---        | :---              | :--- | :---     | :---            | :---       | :---       |:---       |:---                |:---         
| **Pi0.5**   | 100%              | 100% | 100%     | 40%             | 60%        | 10%        | -%        | 80%                | 10%
| **SmolVLA** | 40%               | -%   | -%       | -%              | -%         | -%         | -%        | -%                 | -%
| **GR00T**   | -%                | -%   | -%       | -%              | -%         | -%         | -%        | -%                 | -%
| **X-VLA**   | -%                | -%   | -%       | -%              | -%         | -%         | -%        | -%                 | -%
| **VLA-0**   | -%                | -%   | -%       | -%              | -%         | -%         | -%        | -%                 | -%
| **ACT**     | -%                | -%   | -%       | -%              | -%         | -%         | -%        | -%                 | -%


ACT is the one non-VLA model we tested, and we can see ______ compared to the VLAs

#### Pi0.5

<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/model_c_test_constrained.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 in normal conditions.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_lamp.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with only a background lamp for lighting.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_grey_mat.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with a grey mat instead of wooden worksurface.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_no_wrist_cam.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with the cap blocking the wrist camera.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_orange_arm.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with an orange SO101 arm.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_grey_mat_lamp.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with background lamp lighting and a grey mat worksurface.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_disco.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with a disco light distractor.</p>
</div>
<div style="border: 1px solid #ddd; padding: 12px; border-radius: 8px; margin: 20px 0;">
  <video controls width="100%">
    <source src="https://huggingface.co/mattpidden/pi05_5k_precision-multicolour_block_pick_place/resolve/main/pi05_world_cam_moved.mp4" type="video/mp4">
  </video>
  <p style="color: #666; font-size: 0.9em; margin: 0;">Evaluation of Pi0.5 with a large shift in the world camera poisition and angle.</p>
</div>
