---
title: "How Much Can You Change Before VLAs Break? Testing Environment Robustness"
date: 2026-04-27
author: justintiensmith & mattpidden
excerpt: We broke the setup in every way we could (lighting, cameras, surfaces) and measured what actually mattered. VLAs turned out to be far more robust than expected.
---

## Context

Everyone claims VLAs are great because they generalise. We put it to the test.

| Model   | Normal Conditions | Lamp       | Spotlight | Worksurface    | No wrist camera | Orange arm |
|:--------|:------------------|:-----------|:----------|:---------------|:----------------|:-----------|
| Pi0.5   | 100%              | 100%       | -%        | 100%           | -%              | 60%        |
| SmolVLA | -%                | -%         | -%        | -%             | -%              |            |
| ACT     | -%                | -%         | -%        | -%             | -%              |            |
| Model D | -%                | -%         | -%        | -%             | -%              |            |

