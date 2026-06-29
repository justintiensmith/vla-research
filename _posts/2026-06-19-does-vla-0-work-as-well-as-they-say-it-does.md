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
- i collected a 400 episode real world teleoperaed demo dataset on my SO101 arm for 4 tasks, each 100 episodes that replicate the VLA-0 papers setup and task distribution and type.
- i then tried taking the published LIBERO weights model and doing LoRA fine tuning on those weights with my 400 episode real world dataset (this worked a little, acheiving 10-20% success rates on the real world tasks)
- to take this further i tried doing a full fine tuning using LIBERO weights as starting point on the 400 episodes real world dataset (in progress)
- i then got access to Ismbard AI which has H200 gpus.
- i trained full fine tuning from scratch on full LIBERO dataset to see if it would acehive the results they said (in progress)
- i also full fine tuned from scratch on my 400 real world episode dataset (in progress)
- the next thing i want to try is to add motor state values into the model input (so it doesn't have to guess where it currently is)
- i also want to try fine tuning on the large SO101 dataset collated by Molmmo act and then fine tuning those weights for much less time on my dataset
- eventually im interested in trying to include human demos in the dataset too
