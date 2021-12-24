---
layout: post
title: "Battle of Lines"
---

Don't you think its fascinating how lines and circles are rendered on pixels?
I've been wanting to do this for a while now, to code from scratch Brehensam and Wu's line algorithm in Golang.

<!-- # Table of Contents
1. [Introduction](#Introduction)
2. [Good ol' Brehensam](#Goodol'Brehensam)
3. [Anti-aliasing you say?](#Anti-aliasing you say?) -->

## Introduction

Rasterization turns out to be one of those things that are really old and widely used, yet it is among the least understood rendering technique among most people who rely on it daily (I'm looking at you dear reader). Perhaps let's start with a problem that Rasterization actually solves.
Note: ([For Rastafari, please refer to this link](https://en.wikipedia.org/wiki/Rastafari))

Let's start with drawing a line from a center of one grid (point A) to another (point B) on a pixelized screen. What would have been a simple affair is now something that we have to wrangle with: How can we represent lines between arbitrary points as pixels?

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/line_ab_raw.png" alt="" width="300"/>

__Figure 1: A simple line AB__

Even if we have all the time in the world and choose to manually colorize pixels to achieve the perfectly rasterized line, we might not end up with a set of pixels that best represent the line AB.

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/line_ab_raster_attempt.png" alt="" width="300"/>

__Figure 2: An attempt at rasterizing line AB__

Let's compare this to actually using a rasterization algorithm (Brehensam's Line Algorithm). But the question remains: How do we know what if the rasterized result is actually the closest approximation/representation of the actual line? On what mathematical basis can we decide this? 

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/line_ab_raster_correct.png" alt="" width="300"/>

__Figure 3: Line AB rasterized using Brehensam's Algorithm__


## Good ol' Brehensam
Brehensam has been around from pretty much the beginning of modern computer science. It was developed in 1962 at IBM [1] for a [Calcomp Plotter](https://en.wikipedia.org/wiki/Calcomp_plotter). 

There are plenty of articles out there explaining the Brehensam Line Algorithm and there is a [particular one I like](https://www.cs.helsinki.fi/group/goa/mallinnus/lines/bresenh.html). I think he/she did a hella good job but I would like to build upon Colin's work and make it more visually exciting.

Some articles tend to take put the origin (0,0) at the top left of the image, with the positive x direction towards the right and the positive y direction towards the bottom.

I will avoid using such a convention so we can have more visual intuitiveness. Instead, I will take the following as our coordinate frame: 
<show frame of x and y >

### The 8 Octants
Show diagram for 8 octants
Give pseudocode for each octant


### A slight problem for implementing brehensam for drawing images 
However, we have a slight problem.
Conventionally, image coordinate frames have their origin at the top left, with the x direction being positive towards the right, and y direction towards the bottom.
 
We will need to vertically flip the 8 octants for our model to work

## Can I has a house?

<Show the house I drew>

## What about circles? Can I have them too?

Brehensam has been extended to be drawn for circles too

## Anti-aliasing you say? 

However, some of the issues that Brehensam face is the aliasing effect. The lines appear jagged up close and people get sick of retro-looking graphics after a while. 

Talk about Wu's algorithm and lead up to next article




## Notes
- Diagrams were made with [excalidraw](https://excalidraw.com/)

## References
1. [Paul E. Black. Dictionary of Algorithms and Data Structures, NIST.](https://xlinux.nist.gov/dads/HTML/bresenham.html)