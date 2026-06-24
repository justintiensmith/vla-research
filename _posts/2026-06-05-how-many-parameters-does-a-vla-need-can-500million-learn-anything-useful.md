---
title: "How many parameters does a VLA need? Can 500million learn anything useful?"
date: 2026-06-05
author: mattpidden
excerpt: A review of the VLA-0-Smol paper, which took the VLA-0 idea and experimented on many aspects of it to create a sub-billion parameter VLA.
---

## Introduction

In this blog I take a look at the article [VLA-0-Smol: A Reproducible Recipe for High-Performance, Sub-Billion Parameter VLA](https://robot-learning-collective.github.io/vla-0-smol.html) from The Robot Learning Collective, released in October 2025, with open-sourced code and weights for a LIBERO model. They also have a [GitHub repo](https://github.com/Robot-Learning-Collective/lerobot-experiments).

## Background

This week's blog post is built off the base that [last weeks blog](https://justintiensmith.github.io/vla-research/2026/05/29/why-are-vlas-so-complicated-could-the-simplest-solution-work.html) about VLA-0. This research aims to use the same simple ideas as VLA-0 but with a much smaller VLM backbone. As a reminder, VLA-0 questioned why we would such complicated architectures for VLAs, and if the simplest approach or having the VLM backbone output the actions as a list of integers using text tokens could work. They showed in their results that this simple method acheived results on par with other state of the art methods. 

## The Main Idea

The authors of VLA-0-Smol wanted to create the most acceisslbe VLA, in terms of model size, but also ease of training, understanding and modifying. The most notable contribution is the replacement of Qwen-VL-2.5 model from VLA-0 with SmolVLM2-500M. The authors also integrated VLA-0-Smol into their fork of LeRobot and peroformed detailed ablation studies for all their design choices. The authors state the only change they made to SmolVLM2-500m was to remove the image tiling straegy and just pass in the original image to the encoder to reduce computation. 

## Results

![VLA-0-Smol LIBERO Results](https://github.com/justintiensmith/vla-research/blob/main/images/vla-0-smol-libero-results.png?raw=true)

The results (credit to the original article) show that whilst VLA-0-Smol does acheive slightly lower average score on LIBERO, their performance remains state of the art, out performing policies from much larger research labs. They also significantly outperform all policies that are also sub-billion parameter including Diffusion policy and SmolVLA from the LeRobot team. The authors did note the limits of the LIBERO benchmark and state that their high performance does not necissarliy prove VLA-0-Smol is a capable model for real world use, and they plan to explore this further, however as of writing they have not published anything else on this.


## Ablation Tests

Learning rate sensitivity: 5×10⁻⁶, 1×10⁻⁵, 5×10⁻⁵, 1×10⁻⁴.
Vision encoder fine-tuning vs. freezing.
State/action representations: absolute vs. relative; with/without state tokens.
System prompts: whether structured prompts help after fine-tuning.


## Conclusion
