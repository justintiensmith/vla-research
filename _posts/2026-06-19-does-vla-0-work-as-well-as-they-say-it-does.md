---
title: "Does VLA-0 work as well as they say it does?"
date: 2026-06-19
author: mattpidden
excerpt: I try to reproduce the results in both simulation and real-world from the VLA-0 paper by NVIDIA.
---

## Notes

- i first verified the published LIBERO weights by evaluating on LIBERO and the results match those of the paper.
- I then attempt fine tuning on 200 real world episodes but no real movement, which might have been because scene changed layout a bit during inference compared to the scene camera positions etc of dataset and also could be training configs.
- initally trying to reproduce libero results:
- wanted to a quick eval because full fine tune on all episodes 1.6k of libero would take about 2 weeks on my setup
- `libero_subset` tried first with just 3 libero spatial tasks which totalled 116 episodes and trained with LoRA instead of full tine tuning, loss got down to 0.0 after 50 epochs but the evaluation got 0% and the eval rollouts showed no meaningful movement, just small drift from the robot arms initial positioning.
- `libero_spatial` so i switched to full fine tuning (not lora) on the entire libero-spatial suite (a bit over 400 episodes) and loss graph included looked like it might have learnt but got 0% of full libero spatial rollout eval and again movement was slow, small movements with no direction or purpose, just small drift from initial position.
- i think the problem was some sort of tokenizser setting flag `` but it might also just have been the small amount of data or not enough training steps/epochs.
- to test it for once and all i did full libero full fine tuning using the github repos original settings and everything (still training)
- i still wanted to test what would be good training params for just doing 400 episodes (but to speed up trial and error on real world data i used 400 libero tasks). i did `libero_object` full fine tuning and result was (still in progress)
- whilst these 2 were training i collected a 400 episode real world teleoperaed demo dataset on my SO101 arm for 4 tasks, each 100 episodes that replicate the VLA-0 papers setup and task distribution and type.
- i then tried taking the published LIBERO weights model and doing LoRA fine tuning on those weights with my 400 episode real world dataset (still in progress)
