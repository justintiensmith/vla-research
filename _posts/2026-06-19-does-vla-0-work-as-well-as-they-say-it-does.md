---
title: "Does VLA-0 work as well as they say it does?"
date: 2026-06-19
author: mattpidden
excerpt: I try to reproduce the results in both simulation and real-world from the VLA-0 paper by NVIDIA.
---

## Context



## The Setup


## Notes

- first attempt was fine tuning on 200 real world episodes but no real movement, which might have been because scene changed layout a bit etc
- initally trying to reproduce libero results:
- wanted to a quick eval because full fine tune on all episodes 1.6k of libero would take about a week on my setup
- `libero_subset` tried first with just 3 libero spatial tasks which totalled 116 tasks and trained with LoRA instead of full tine tuning, loss got down to 0.0 after x number of epochs but the evaluation got 0% and the eval rollouts showed no meaningful movement
- `libero_spatial` so i switched to full fine tuning (not lora) on the entire libero-spatial suite (a bit over 400 episodes) and loss graph included looked like it might have learnt but got 0% of full libero spatial rollout eval and again movement was slow, small movements with no direction or purpose.
