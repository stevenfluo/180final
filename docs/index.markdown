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

[Prof. Ren Ng et al.](http://graphics.stanford.edu/papers/lfcamera/lfcamera-150dpi.pdf) demonstrated that capturing images over a plane orthogonal to the optical axis (for example, along the image/sensor plane) allows one to create complex effects using very simple processing. These images of the same scene captured from different angles of view allow for post-capture refocusing and aperture resizing. Having lightfield data is especially useful for photographers because they no longer have to trade off controlling depth of field to control motion blur, and vice versa. My goal was to recreate these effects using real lightfield data.

I used the rectified images provided in the [Stanford Light Field Archive](http://lightfield.stanford.edu/lfs.html), which has sample datasets comprising of multiple images taken over a regularly spaced grid. I use alignment and averaging methods to implement depth refocusing and aperture adjustment on the sample data.

Note: if the GIFs don't play properly, reload or open the GIF in a new tab.

# Part 1.1: Depth Refocusing

The objects which are far away from the camera do not vary their position significantly when the camera moves around while keeping the optical axis direction unchanged. The nearby objects, on the other hand, vary their position significantly across images. Averaging all the images in the grid without any shifting will produce an image which is sharp around the far-away objects but blurry around the nearby ones. Similarly, shifting the images "appropriately" and then averaging allows one to focus on object at different depths. 

Each sub-aperture image in the Stanford dataset can be seen as the light from a different part of a "universal aperture" (or the same "back of the lens" picture taken many times from slightly different positions). These images can be thought of as light rays because each (x, y) position on the image plane and (u, v) position on the aperture plane uniquely define a ray. In the dataset, the (x, y) is fixed while (u, v) is varied. This gives us a set of rays that will be integrated by a single point on the image plane, which we repeat for every point on the image plane.

Using this idea, I implement a function to generate images focusing on different depths. After extracting sub-aperture positional information by parsing the filenames of images in the dataset, I determined the position of the center microlens. Since the Stanford Light Field Archive datasets I used have a 17x17 microlens array, I used (8, 8) as the center. I then implemented my depth refocusing algorithm as follows (with guidance from past lecture slides):

1. For each sub-aperture image I(x, y):
    1. Calculate the displacement (u, v) from the center.
    2. Shift the sub-aperture image by C * (u, v), where C is an arbitrary scaling factor. If shifts are not integers, bilinear interpolation may be needed.
2. Average all of the shifted images.

Note that a larger C means refocusing further from the physical focus, and the sign of C affects whether we are focusing closer or further.

I initially used `np.roll` to actually shift the image, but I switched to `scipy.ndimage.shift` so allow for non-integer shifts when using fractional values of C. Because of system limitations and image sizes, I used `order = 1` (linear) as the interpolation parameter because the visual difference between images interpolated at `order = 1` and `order = 3` was very minimal, with an *~8x speedup*. 

<p align="center">
    <img src="./img/ordertest0.jpg" alt="ad" width="22%"/>
    <img src="./img/ordertest1.jpg" alt="ad" width="22%"/>
    <img src="./img/ordertest2.jpg" alt="ad" width="22%"/>
    <img src="./img/ordertest3.jpg" alt="ad" width="22%"/>
    <p style="text-align: center;"><i>The same image interpolated using order 0, 1, 2, and 3. The significant performance improvement justifies the *slight*, hardly noticeable decrease in interpolation quality.</i></p>
</p>

Here are my results on different datasets:

<p align="center">
    <img src="./img/chess_depth_withreverse.gif" alt="ad" width="95%"/>
    <p style="text-align: center;"><i>Chess; generated with evenly spaced C = -0.5 to 3.0</i></p>
</p>

<p align="center">
    <img src="./img/flower_depth_withreverse.gif" alt="ad" width="95%"/>
    <p style="text-align: center;"><i>Eucalpytus flowers; generated with evenly spaced C = -0.5 to 3.0</i></p>
</p>

<p align="center">
    <img src="./img/stone_depth_withreverse.gif" alt="ad" width="95%"/>
    <p style="text-align: center;"><i>Amethyst; generated with evenly spaced C = -1.5 to 3.0</i></p>
</p>

<p align="center">
    <img src="./img/truck_depth_withreverse.gif" alt="ad" width="95%"/>
    <p style="text-align: center;"><i>Lego Bulldozer; generated with evenly spaced C = -1.5 to 3.0</i></p>
</p>

# Part 1.2: Aperture Adjustment

Averaging a large number of images sampled over the grid perpendicular to the optical axis mimics a camera with a much larger aperture, and using fewer images results in an image that mimics a smaller aperture. 

To change the aperture, I modified my shift function to add an aperture threshold in order to reduce the number of images we use in our averaging, and also to reduce the shift of the images we do choose to include. The aperture threshold sets the maximum radius from the center that we will include in our average image. If the threshold is large, we'd average over many images and create a larger aperture, or a shallow depth of field.

A brief note about noise: having more sub-aperture images in the result may introduce more noise/blur because of the distance between sub-apertures and their slight difference in views.

Here are my results on different datasets. For all results, I use an aperture threshold of 0 to 10. The value of C is held constant across these images and is specified in the caption of each output.

<p align="center">
    <img src="./img/chess_aperture_withreverse.gif" alt="ad" width="95%"/>
    <p style="text-align: center;"><i>Chess; C = 1.0 </i></p>
</p>

<p align="center">
    <img src="./img/flower_aperture_withreverse.gif" alt="ad" width="95%"/>
    <p style="text-align: center;"><i>Eucalpytus flowers; C = 2.5</i></p>
</p>

<p align="center">
    <img src="./img/stone_aperture_withreverse_final.gif" alt="ad" width="95%"/>
    <p style="text-align: center;"><i>Amethyst; C = 0.8</i></p>
</p>

<p align="center">
    <img src="./img/truck_aperture_withreverse.gif" alt="ad" width="95%"/>
    <p style="text-align: center;"><i>Lego Bulldozer; C = 2.5</i></p>
</p>

<p align="center">
    <img src="./img/ball_aperture_withreverse.gif" alt="ad" width="95%"/>
    <p style="text-align: center;"><i>Crystal Ball; C = 0.1</i></p>
</p>


# Part 1.3: Summary

I was impressed by how the simple idea of placing multiple sensors, combined with elementary operations, can lead to really interesting results. I looked up more information about the Lytro camera (R.I.P.) and it made me interested in actually getting on to experiment. I learned a lot about computational photography through this project, and definitely feel like I have a better grasp on how to manipulate rays and the nuances of depth focusing and aperture. If I had more time, I would've tried to set up my own setup to take photos simulating different rays in the same scene to try to create my own dataset.


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