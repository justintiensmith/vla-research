---
title: "How Much Can You Change Before VLAs Break? Testing Environment Robustness"
date: 2026-04-27
author: justintiensmith & mattpidden
excerpt: We stress-tested our VLA setup by changing the lighting, worksurface, and camera configuration. The results? VLAs are ______.
---

## The Generalisation Claim

One of the primary selling points of VLAs is their ability to generalise across diverse environments. But "generalisation" is often a vague term used in papers; this week, we decided to experiment with VLA generalisation to diverse environments.

## The Experiment

All of the models we selected were trained on the same dataset to ensure a fair comparison. We used the dataset that provided the best performance across various model architectures in our previous experiment. You can find the dataset here: [lerobot/red_block_precision-multicolour_block_pick_place](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fjustintiensmith%2Fred_block_precision-multicolour_block_pick_place%2Fepisode_199).

The dataset consists of 200 episodes:
* **Precision Skill (100 episodes):** Picking up a red block within a defined 10cm x 10cm square area.
* **Diversity Skill (100 episodes):** Picking up blocks of different colors in a range of locations around the workspace.

For our experiments, we established a **Control Experiment** where we ran inference 10 times in the same setup as the data collection: a standard workspace table, fluorescent lab lighting, and a fixed world camera position. 

We defined success as completing the task within 60 seconds. To keep the testing focused, we only placed the block within the 10cm x 10cm "precision" square and used only the red block for all inference runs.

### Results

| Model       | Normal Conditions | Lamp | Grey Mat | No Wrist Camera | Orange Arm | Mat + Lamp | Paper Cup | Lamp + Disco Light | Shifted World Camera
| :---        | :---              | :--- | :---     | :---            | :---       | :---       |:---       |:---                |:---         
| **Pi0.5**   | 100%              | 100% | 100%     | 40%             | 60%        | 10%        | -%        | 80%                | -%
| **SmolVLA** | -%                | -%   | -%       | -%              | -%         | -%         | -%        | -%                 | -%
| **GR00T**   | -%                | -%   | -%       | -%              | -%         | -%         | -%        | -%                 | -%
| **X-VLA**   | -%                | -%   | -%       | -%              | -%         | -%         | -%        | -%                 | -%
| **VLA-0**   | -%                | -%   | -%       | -%              | -%         | -%         | -%        | -%                 | -%
| **ACT**     | -%                | -%   | -%       | -%              | -%         | -%         | -%        | -%                 | -%


ACT is the one non-VLA model we tested, and we can see ______ compared to the VLAs


