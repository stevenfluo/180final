---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
title: Home
permalink: /
---

<style>
        .grid-container {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 20px;
            text-align: center;
        }
        .grid-item {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .grid-item img {
            width: 100%;
            height: auto;
        }
        .grid-item p {
            font-style: italic;
        }
</style>

<!-- MathJax -->
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

# **Project 1: Lightfield Camera**

# Overview


# Summary


# **Project 2: High Dynamic Range Imaging**

# Overview

In this project, I experimented with diffusion models, implemented diffusion sampling loops, and used them for tasks including inpainting and creating optical illusions.

# Part A.1: Setup

I used Stability AI's DeepFloyd IF two-stage diffusion model. The first stage produces images of size 64x64 and the second stage takes the outputs of the first stage and generates images of size 256x256.

To test the model, I used the sample captions to generate images. I use a random seed of 180 throughout the project.

Here are the stage 1 and stage 2 outputs when `num_inference_steps = 20`.

<p align="center">
    <img src="./img/Unknown.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-1.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-2.png" alt="ad" width="30%"/>
    <p style="text-align: center;"><i>Stage 1 — 1. "an oil painting of a snowy mountain village", 2. "a man wearing a hat", 3. "a rocket ship"</i></p>
</p>

<p align="center">
    <img src="./img/Unknown-3.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-4.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-5.png" alt="ad" width="30%"/>
    <p style="text-align: center;"><i>Stage 2 — 1. "an oil painting of a snowy mountain village", 2. "a man wearing a hat", 3. "a rocket ship"</i></p>
</p>

Here are the stage 1 and stage 2 outputs when `num_inference_steps = 50`.

<p align="center">
    <img src="./img/Unknown-6.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-7.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-8.png" alt="ad" width="30%"/>
    <p style="text-align: center;"><i>Stage 1 — 1. "an oil painting of a snowy mountain village", 2. "a man wearing a hat", 3. "a rocket ship"</i></p>
</p>

<p align="center">
    <img src="./img/Unknown-9.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-10.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-11.png" alt="ad" width="30%"/>
    <p style="text-align: center;"><i>Stage 2 — 1. "an oil painting of a snowy mountain village", 2. "a man wearing a hat", 3. "a rocket ship"</i></p>
</p>

# Part A.2: Sampling Loops

# Part A.2.1: Implementing the Forward Process

I use sampling loops to generate images from the diffusion model through an iterative denoising process: starting form pure noise at timestep T (sample from a Gaussian distribution), we can predict and remove part of the noise, repeating this process until we arrive at a clean image. DeepFloyd models do this over 1000 timesteps.

The forward process adds noise to a clean image from a Gaussian distribution with a specific mean and variance at each timestep.

`alphas_cumprod` is the hyperparameter denotes the noise level, where smaller t values correspond to cleaner images.

The function forward(im, t) produces a noised image at step t.

<p align="center">
    <img src="./img/Unknown-12.png" alt="ad" width="22%"/>
    <img src="./img/Unknown-13.png" alt="ad" width="22%"/>
    <img src="./img/Unknown-14.png" alt="ad" width="22%"/>
    <img src="./img/Unknown-15.png" alt="ad" width="22%"/>
    <p style="text-align: center;"><i>Four views of the Campanile: no noise, noisy at t=250, noisy at t=500, noisy at t=750.</i></p>
</p>

## Part A.2.2: Classical Denoising

I use Gaussian blur filtering to denoise the noised images.

<p align="center">
    <img src="./img/Unknown-13.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-14.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-15.png" alt="ad" width="30%"/>
    <p style="text-align: center;"><i> Noisy at t=250, noisy at t=500, noisy at t=750.</i></p>
</p>

<p align="center">
    <img src="./img/Unknown-16.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-17.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-18.png" alt="ad" width="30%"/>
    <p style="text-align: center;"><i> Gaussian blur denoising at t=250, denoising at t=500, denoising at t=750.</i></p>
</p>

## Part A.2.3: One-Step Denoising

For one-step denoising, I used the UNet to denoise the image by estimating the noise. First, I estimated the noise in the new noisy image, by passing it through `stage_1.unet`, which I removed from the noisy image to estimate the original one.

<p align="center">
    <img src="./img/Unknown-13.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-14.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-15.png" alt="ad" width="30%"/>
    <p style="text-align: center;"><i> Noisy at t=250, noisy at t=500, noisy at t=750.</i></p>
</p>

<p align="center">
    <img src="./img/Unknown-19.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-20.png" alt="ad" width="30%"/>
    <img src="./img/Unknown-21.png" alt="ad" width="30%"/>
    <p style="text-align: center;"><i> One-step denoising at t=250, denoising at t=500, denoising at t=750.</i></p>
</p>

## Part A.2.4: Iterative Denoising

Diffusion models perform better when iteratively denoising images — that's how they were designed! Even though I want to iteratively denoise my noisy images across 1000 timesteps, I skip steps to speed things up, using `strided_timesteps` to iteratively take small strided timesteps in order to produce a clean image.

<p align="center">
    <img src="./img/Unknown-22.png" alt="ad" width="15%"/>
    <img src="./img/Unknown-23.png" alt="ad" width="15%"/>
    <img src="./img/Unknown-24.png" alt="ad" width="15%"/>
    <img src="./img/Unknown-25.png" alt="ad" width="15%"/>
    <img src="./img/Unknown-26.png" alt="ad" width="15%"/>
    <img src="./img/Unknown-27.png" alt="ad" width="15%"/>
    <p style="text-align: center;"><i> Iteratively denoising: iteration 10, t=690; iteration 15, t=540; iteration 20, t=390; iteration 25, t=240; iteration 30, t=90; fully denoised.</i></p>
</p>

<p align="center">
    <img src="./img/Unknown-12.png" alt="ad" width="22%"/>
    <img src="./img/Unknown-27.png" alt="ad" width="22%"/>
    <img src="./img/Unknown-28.png" alt="ad" width="22%"/>
    <img src="./img/Unknown-29.png" alt="ad" width="22%"/>
    <p style="text-align: center;"><i> Original image, iteratively denoised, one-step denoised, and Gaussian blur denoised images. </i></p>
</p>

Iterative denoising clearly performs better than the other methods!


## Part A.2.5: Diffusion Model Sampling

Using the `iterative_denoise` function I implemented, I can also generate images from scratch! I do this by setting `i_start = 0` and passing in random noise (drawn from a Gaussian distribution) — essentially denoising pure noise. This method and the prompt "a high quality photo" yields these sampled images:

<p align="center">
    <img src="./img/Unknown-43.png" alt="ad" width="18%"/>
    <img src="./img/Unknown-44.png" alt="ad" width="18%"/>
    <img src="./img/Unknown-32.png" alt="ad" width="18%"/>
    <img src="./img/Unknown-33.png" alt="ad" width="18%"/>
    <img src="./img/Unknown-34.png" alt="ad" width="18%"/>
    <p style="text-align: center;"><i> Images generated from pure noise. </i></p>
</p>

Here's an example of the denoising process, visualized with intermediate images:
<p align="center">
    <img src="./img/Unknown-35.png" alt="ad" width="11%"/>
    <img src="./img/Unknown-36.png" alt="ad" width="11%"/>
    <img src="./img/Unknown-37.png" alt="ad" width="11%"/>
    <img src="./img/Unknown-38.png" alt="ad" width="11%"/>
    <img src="./img/Unknown-39.png" alt="ad" width="11%"/>
    <img src="./img/Unknown-40.png" alt="ad" width="11%"/>
    <img src="./img/Unknown-41.png" alt="ad" width="11%"/>
    <img src="./img/Unknown-42.png" alt="ad" width="11%"/>
    <p style="text-align: center;"><i> Iteratively denoising: iteration 0, t=990; iteration 5, t=840; iteration 10, t=690; iteration 15, t=540; iteration 20, t=390; iteration 25, 5=240; iteration 30, t=90; denoised image.</i></p>
</p>