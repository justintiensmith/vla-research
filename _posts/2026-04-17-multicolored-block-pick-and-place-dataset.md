---
title: "Multicolored-Block-Pick-And-Place-Dataset"
date: 2026-04-17
---

Today, we collected a new pick-and-place dataset with 200 episodes using four different colored blocks. Most of the episodes were collected by picking up the object on the left and placing it in a bin on the right.

Our first “toy” dataset contained only 50 episodes and used an apple as the object to pick and place in a ceramic bowl. The apple barely fit in the robot gripper, which made the setup less reliable. The world camera in that dataset was also mounted directly above the workspace, looking straight down. We suspected that this initial setup was not ideal because the dataset was quite small, and the camera angle likely did not capture depth information well enough.

To improve on that first dataset, we made three major changes. First, we repositioned the world camera so that it viewed the workspace from an angle, looking down from the front of the robot rather than directly overhead. Second, we replaced the apple with Duplo LEGO blocks, which fit much more naturally in the SO-101 gripper. Third, we collected 200 episodes instead of just 50.

The new dataset includes four block colors, with each color picked up 50 times. Of those 50 episodes, 25 involve only the target block, while the other 25 include distractor blocks. Within those distractor episodes, 10 include one additional block, 10 include two additional blocks, and 5 include three additional blocks.

<img src="{{ '/media/red-green-blue-yellow-blocks.JPG' | relative_url }}" alt="Our multicolored blocks" width="600">

Each episode is paired with a language instruction corresponding to the target color. For example:

**Language instruction:**  
“Pick up the {blue, red, green, yellow} block and carefully place it in the black bin.”

Our dataset is available on Hugging Face here:  
[View the dataset](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fjustintiensmith%2Fmulticolour_block_pick_place%2Fepisode_0)
