---
title: "Why are VLAs so Complicated? Could the Simplest Solution Work?"
date: 2026-05-29
author: mattpidden
excerpt: A look at VLA-0, a surprisingly simple approach to robotic control that uses a standard vision-language model with no architectural changes, from NVIDIA Research.
---

## Introduction

In this blog I take a look at the paper [VLA-0: Building State-of-the-Art VLAs with Zero Modification](https://arxiv.org/pdf/2510.13054) from NVIDIA Research, released in October 2025, with open-sourced code and weights for a LIBERO model. They also have a [project website](https://vla0.github.io/) and a [GitHub repo](https://github.com/NVlabs/vla0).

## Background

As I have written about previously, a Vision Language Action (VLA) model is an end-to-end policy for robotics. It takes as input frames from the robot's cameras and a language prompt, and outputs actions.

Many variations of VLAs exist, with some also taking the robot’s current state as input, or a depth map, etc. These models are mostly built on top of existing trained Vision Language Models (VLMs), which are popular and well known thanks to ChatGPT, Claude, Gemini, etc. They take vision and language inputs (like uploading a photo of your house with a prompt asking for suggestions for improvements), and the model returns a text output. Some models also support generating images.

Whilst these models may appear very similar to an end user, the architecture and training methods vary greatly. The VLA-0 paper broadly categorises these into three approaches.

The first is discrete token VLAs. This approach was used by some of the earliest VLAs, including RT-2 (the first VLA). Robot actions are naturally continuous, but in this case they are discretised into a number of bins. Each bin is then assigned a token. Some models replace existing tokens in the VLM vocabulary, and some insert new tokens into the vocab. These models are then trained using standard VLM training methods, typically cross-entropy loss on robot demonstrations. The paper states these types of models restrict the resolution of actions, since you need thousands of bins, which conflicts with sharing the text vocabulary, and the training method can reduce the understanding the VLM already had.

The next type of VLA the paper describes is generative action head VLAs. The VLM is fine-tuned to predict a latent action vector which is uninterpretable and not directly useful, but doesn’t require any new or replaced tokens. To make the vector useful, another neural network (the action head) is trained to take those latent vectors as input and output usable actions, usually via flow matching or a diffusion process. This is more complicated to train and build, and apparently leads to a decline in language understanding and grounding capabilities of the VLM.

The final type of VLA mentioned is custom architecture VLAs. This is a broad category and basically includes any other type of VLA that does its own custom thing. The paper lists OpenVLA-OFT as an example, which has a specialised ACT head, and also Pi-Fast, which uses a special tokenisation scheme for actions. These custom methods are often state-of-the-art, but involve significant architectural changes, extra parameters, custom pipelines, and are sometimes closed behind company doors.

## The Main Idea

The authors then ask whether there is a simpler approach. They suggest using a VLM to output text directly, as it was originally designed to do. The text would be an integer list corresponding directly to the robot’s actions. They call their model VLA-0 because it has zero modifications from a standard VLM.

In this diagram (credit to the original paper), you can see the architecture of VLA-0. It takes 3 input types (images, system prompt, task instruction) and outputs the robot actions directly. 

![VLA-0 Architecture](https://github.com/justintiensmith/vla-research/blob/main/images/vla-0-architecture-diagram.png)

## Methodology

The authors chose to use the 3-billion parameter Qwen-VL-2.5 model because it has highly competitive performance for its size and is fully open source, which makes it accessible and reproducible. However, they note their method is not Qwen-dependent and could, in theory, work with many different VLMs.

VLA-0 has the same input structure as Qwen, taking a system prompt, images, and a task instruction prompt. During fine-tuning they used the following system prompt, where H, D, and B are set parameters:

> "Analyse the input image and predict robot actions for the next H timesteps. Each action has D dimensions. Output a single sequence of H × D integers (0 to B), representing the H timesteps sequentially. Provide only space-separated numbers. Nothing else."

To simplify the task of outputting actions as text, the authors ask the VLM to output actions as integers. The original continuous actions from the robotic dataset are normalised to a fixed integer range, and the VLM must then generate an integer for each dimension within that range. The exact value of the range can be adjusted; they provide an example of a [0–1000] range. They note that whilst this is discrete, it does not have the same disadvantage as discrete token-based VLAs, where increasing the number of bins begins to damage the VLM’s vocabulary and remove its previous abilities, because increasing the integer range does not add or replace any tokens.

To train the model they perform full fine-tuning of the Qwen VLM (meaning all parameters are updated, in this case approximately 3 billion parameters). Training took 32 hours on 8 A100 GPUs.

## Results

For simulation benchmarks the authors use the well-known LIBERO benchmark, which contains 1.6k demonstration episodes. The model is evaluated across 40 tasks, across four evaluation suites, with 50 rollouts per task (10 tasks per suite). VLA-0 achieves an average 94.7% success rate across the benchmark. It outperforms all models that do not use large-scale action pretraining on each suite and on average. When compared to models that do use large-scale action pretraining, VLA-0 still performs very well, beating many state-of-the-art models, although notably OpenVLA-OFT does achieve a slightly higher average. Nevertheless, these results are very impressive considering the simplicity VLA-0 offers compared to other VLAs. The full LIBERO results are shown in the figure below (credit to original paper).

![VLA-0 LIBERO Results](https://github.com/justintiensmith/vla-research/blob/main/images/vla-0-libero-results.png)

In real-world evaluation, they trained on four tasks with 100 demos per task, for a total of 400 training episodes. The tasks were simple pick-and-place behaviours such as “Place the banana on the plate”. The model performed worse in the real world than in simulation, which is expected behaviour for VLAs, but still outperformed SmolVLA (a VLA from the Hugging Face / LeRobot team), which was exclusively trained on large amounts of SO-100/101 data (the robot embodiment used in evaluation, so in theory SmolVLA had a big advantage). On average, VLA-0 achieved a 60% success rate, whilst SmolVLA achieved 47.5%. These success rates seem reasonable and are in line with our personal experiences described in earlier blogs with the SmolVLA model. The real-world results are shown in the figure below (credit to original paper).

![VLA-0 Real-World Results](https://github.com/justintiensmith/vla-research/blob/main/images/vla-0-real-world-results.png)


## Ablation Tests

When creating the model, the authors performed several ablation tests to justify design choices and improve performance. All ablation results were tested on the LIBERO suite.

The team introduced action ensembling, inspired by the Action Chunking Transformer (ACT), whereby the VLM predicts a sequence of actions n steps into the future. For the current timestep, there are n previous predictions which can be averaged to produce a smoother action. This improved success rate by 2 points.

They also introduced Masked Action Augmentation, which increased success by 1.2 points. This method randomly masks out characters in the target action string during training. This forces the VLM to reason about the action based on its input, instead of simply completing a numerical sequence, which autoregressive models can often exploit.

They experimented with the action range and found that 0–1000 was sufficient. A range of 0–250 reduced performance by 1.5 points, while 0–4000 provided no additional benefit.

Finally, they tested image tiling, where multiple images are either passed separately or combined into a single tiled image. This had no effect on performance.

## Conclusion

Overall, the VLA-0 paper from NVIDIA is a great read: short, simple, and quite inspiring. It reminds us not to overlook simple and sometimes obvious solutions just because they don’t sound like they should work. They also mention future work could explore how VLA-0 performs with large-scale action data (i.e. tens of thousands of demos instead of 400), as well as improving inference speed (currently around 4 Hz). The full paper and results are linked at the top of this blog.

I personally look forward to trying to reproduce their results and building upon this methodology.
