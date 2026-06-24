---
title: "How many parameters does a VLA need? Can 500 million learn anything useful?"
date: 2026-06-05
author: mattpidden
excerpt: A review of VLA-0-Smol, which explores whether sub-billion parameter vision-language-action models can still achieve strong robotic control using a simplified VLA-0 style approach.
---

## Introduction

In this blog I take a look at the article [VLA-0-Smol: A Reproducible Recipe for High-Performance, Sub-Billion Parameter VLA](https://robot-learning-collective.github.io/vla-0-smol.html) from The Robot Learning Collective, released in October 2025, with open-sourced code and weights for a LIBERO model. They also have a [GitHub repo](https://github.com/Robot-Learning-Collective/lerobot-experiments).

## Background

This week's blog post is built on top of the base from [last week’s blog](https://justintiensmith.github.io/vla-research/2026/05/29/why-are-vlas-so-complicated-could-the-simplest-solution-work.html) about VLA-0. This research aims to use the same simple ideas as VLA-0 but with a much smaller VLM backbone. As a reminder, VLA-0 questioned why we use such complicated architectures for VLAs, and whether the simplest approach of having a VLM backbone output actions as a list of integers using text tokens could work. They showed in their results that this simple method achieved results on par with other state-of-the-art methods.

## The Main Idea

The authors of VLA-0-Smol wanted to create the most accessible VLA, in terms of model size, but also ease of training, understanding, and modification. The most notable contribution is the replacement of the Qwen-VL-2.5 model from VLA-0 with SmolVLM2-500M. The authors also integrated VLA-0-Smol into their fork of LeRobot and performed detailed ablation studies for all their design choices. They state the only change they made to SmolVLM2-500M was removing the image tiling strategy and instead just passing the original image into the encoder to reduce computation.

## Results

![VLA-0-Smol LIBERO Results](https://github.com/justintiensmith/vla-research/blob/main/images/vla-0-smol-libero-results.png?raw=true)

The results (credit to the original article) show that while VLA-0-Smol does achieve a slightly lower average score on LIBERO, its performance remains state-of-the-art, outperforming policies from much larger research labs. It also significantly outperforms all other sub-billion parameter policies, including Diffusion Policy and SmolVLA from the LeRobot team.

The authors do note the limits of the LIBERO benchmark and state that high performance there does not necessarily prove VLA-0-Smol is a capable model for real-world use. They plan to explore this further, although as of writing they have not published anything additional on this.

## Ablation Tests

The authors test learning rate sensitivity: 5×10⁻⁶, 1×10⁻⁵, 5×10⁻⁵, and 1×10⁻⁴. They find that learning rate has a large impact and should be treated more seriously than a hyperparameter that can just be casually tuned. They observe that the smallest learning rates (5×10⁻⁶ and 1×10⁻⁵) both fail to learn anything, while 5×10⁻⁵ and 1×10⁻⁴ achieve similar success rates.

They also investigate whether freezing the vision encoder affects performance. The benefit is that freezing reduces training cost. They note that vision encoders are typically trained on natural images, not robot scenes. However, they find that freezing the encoder significantly drops success rate, so it is important to fine-tune it for this architecture. They also mention their disappointment from a practical standpoint, since freezing would have allowed faster training.

They perform ablations on action representations, testing absolute actions, relative actions, and including a state vector. However, it is important to note that this (and all ablation studies in the paper) is done on a simple 2D Push-T task rather than LIBERO or real-world settings, so these results may not generalise across domains. They also note they could not test relative actions on LIBERO due to implementation issues.

Finally, the authors test whether the system prompt used in the original VLA-0 has any effect on performance. From a personal perspective, I like its inclusion, as it gives the feeling of a complete VLM understanding its task simply by being instructed to act as a VLA. However, the ablation shows no difference in performance. This is expected, since the model is fine-tuned for tens or hundreds of thousands of steps, meaning the weights learn to output the correct action format purely through the loss.

## Conclusion

While the authors of VLA-0-Smol show that 500 million parameters can achieve state-of-the-art results on benchmarks like LIBERO, these initial findings are still very encouraging. However, it is hard to say for sure whether a 500 million parameter model is sufficient for strong reasoning, broad generalisation, and real-world performance without more extensive quantitative results.
