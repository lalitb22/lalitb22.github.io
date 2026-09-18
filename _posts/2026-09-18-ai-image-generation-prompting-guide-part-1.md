---
title: "Image Generation: How to Write Effective Prompts - Part 1"
date: 2026-09-18 12:00:00 +0000
categories: [Artificial Intelligence, Prompt Engineering]
tags: [ai image generation, text to image, prompt guide, midjourney, stable diffusion, digital creation]
description: "A standard, fluff-free guide to structuring AI image prompts. Learn core formulas for product ads, portraits, and landscape photography."
pin: false
math: false
mermaid: false
---

Getting reliable, high-quality results from AI image generators comes down to one standard workflow: **describe the physical elements of the scene** rather than padding your input with generic quality buzzwords.

---

## Core Best Practices

* **Lead with the subject:** Generative image models prioritize early tokens. Always define the primary subject in your first 3 to 6 words before specifying secondary elements.
* **Skip subjective quality words:** Remove terms like *“photorealistic,”* *“4K,”* or *“stunning.”* Instead, specify concrete physical attributes—such as *fine water droplets*, *matte ceramic finish*, or *woven knit fabric*.
* **Specify light sources:** Flat lighting makes AI images look synthetic. Anchor your image with explicit sources: *soft morning window light*, *overhead softbox studio lighting*, or *golden hour backlighting*.
* **Define camera framing:** Prevent unwanted cropping by setting the view angle up front: *macro shot*, *eye-level close-up*, or *wide-angle landscape*.

---

## Prompt Frameworks by Category

Different outputs need different structural cues to guide the model:

| Category | Key Variables to Include | Production-Ready Prompt |
| :--- | :--- | :--- |
| **Product Ad** | Plain background, crisp edges, clean studio lighting, copy space for typography | `Product photo of a matte black ceramic mug on a polished concrete surface, overhead softbox studio lighting, neutral grey background, crisp focus, copy space on top right.` |
| **Portrait** | Subtle skin texture, eye-level framing, shallow depth of field | `Eye-level portrait of a smiling woman in her 30s wearing a neutral linen shirt, natural diffused window light from the left, softly blurred living room background.` |
| **Landscape** | Foreground anchor point, atmospheric haze, explicit time of day | `Wide-angle view of jagged pine mountains at sunrise, rocky hiking path in the foreground, warm sunbeams cutting through morning mist.` |

---

## The Universal Prompt Formula

When starting from a blank prompt box, chain your parameters using this linear structure:

```text
[Subject] + [Placement & Environment] + [Lighting Source] + [Framing & Shot Type]
```

### Applied Example

> **Prompt:**  
> `Ceramic coffee mug resting on an aged oak table next to a closed notebook, warm morning sunlight streaming from the side, eye-level close-up shot.`

By giving the model physical parameters instead of subjective praise, you eliminate random generations and achieve consistent, repeatable results.
