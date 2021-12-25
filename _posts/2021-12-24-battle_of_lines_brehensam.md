---
layout: post
title: "Battle of Line Rasters - Brehensam"
---

Don't you think its fascinating how lines and circles are rendered on pixels?
I've been wanting to do this for a while now, to code from scratch Brehensam and Wu's line algorithm in Golang.

<!-- # Table of Contents
1. [Introduction](#Introduction)
2. [Good ol' Brehensam](#Goodol'Brehensam)
3. [Anti-aliasing you say?](#Anti-aliasing you say?) -->

# Introduction

Rasterization turns out to be one of those things that are really old and widely used, yet it is among the least understood rendering technique among most people who rely on it daily (I'm looking at you dear reader). Perhaps let's start with a problem that Rasterization actually solves.

Let's start with drawing a line from a center of one grid (point A) to another (point B) on a pixelized screen. What would have been a simple affair is now something that we have to wrangle with: How can we represent lines between arbitrary points as pixels?

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/line_ab_raw.png" alt="A simple line AB" width="300"/>

__Figure 1: A simple line AB__

Even if we have all the time in the world and choose to manually colorize pixels to achieve the perfectly rasterized line, we might not end up with a set of pixels that best represent the line AB.

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/line_ab_raster_attempt.png" alt="An attempt at rasterizing line AB" width="300"/>

__Figure 2: An attempt at rasterizing line AB__

Let's compare this to actually using a rasterization algorithm (Brehensam's Line Algorithm). But the question remains: How do we know what if the rasterized result is actually the closest approximation/representation of the actual line? On what mathematical basis can we decide this? 

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/line_ab_raster_correct.png" alt="Line AB rasterized using Brehensam's Algorithm" width="300"/>

__Figure 3: Line AB rasterized using Brehensam's Algorithm__


# Good ol' Brehensam
Let's take a look at the Brehensam Line Algorithm, a classic line rasterization algorithm still in great use today. It was developed in 1962 at IBM [1] for a [Calcomp Plotter](https://en.wikipedia.org/wiki/Calcomp_plotter). 
There are plenty of articles out there explaining the Brehensam Line Algorithm and there is a [particular one I like](https://www.cs.helsinki.fi/group/goa/mallinnus/lines/bresenh.html). I think he did a hella good job and I would like to build upon Colin's work and focus on the intuition which will lead us to the algorithm.

Small note: Some articles tend to use the following coordinate frame, taking the top left as the origin (0,0), right as positive x direction, and downwards as the positive y direction.

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/image_coordinate_frame.png" alt="Image Coordinate Frame" width="100"/>

__Figure 4: Image Coordinate Frame__

To keep it visually intuitive. I will instead use the graph coordinate frame, and not worry about how it will translate to the image coordinate frame for now: 

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/graph_coordinate_frame.png" alt="Graph Coordinate Frame" width="100"/>

__Figure 5: Graph Coordinate Frame__

## Problem Statement

The problem statement is to represent a line from point A(x1, y1) to B(x2, y2) on a grid. We make the following assumptions:
1. The start and end points coordinate are integers. 
2. We will only be incrementing x as we plot the line. 

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/brehensam_algo/brehensam_0.png" alt="brehensam_0" width="450"/>

__Figure 6: Problem Set up__

Given the following assumption, it suffices to say that we have only 2 possible choices from which to plot. Will it be (x+1, y) in figure 7 or (x+1, y+1) in figure 8?
Brehensam's algorithm will be alternating between these 2 choices for the first octant.

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/brehensam_algo/brehensam_1a.png" alt="brehensam_1a" width="400"/>

__Figure 7: Pixelize (x+1, y)__

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/brehensam_algo/brehensam_1b.png" alt="brehensam_1b" width="400"/>

__Figure 8: Or pixelize (x+1, y+1)?__

However, as we start iterating through the algorithm, we will notice that the center of the plotted grid will have an error offset from the ACTUAL line. This is illustrated in figure 9.
```
#x1 and y1 are starting coordinates of the line
x = x1, y = y1

#Iteration 0
Actual line coordinate = (x, y)
Rasterized grid = (x, y)

#Iteration 1
Actual line coordinate = (x, y + e)
Rasterized grid = (x + 1, y)

#Iteration 2
Actual line coordinate = (x, y + e + m)
Rasterized grid = (x + 2, y + 1)

```

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/brehensam_algo/brehensam_2.png" alt="brehensam_2" width="500"/>

__Figure 9: Error offset of y coordinate__

To determine which grid to actually plot on, we have to focus on the difference between the rasterized y coordinate and the actual line y coordinate. In the example above, we will only plot on (x+2, y+1) if:
> **y + e + m >= y + 0.5**

Similarly, we will only plot on (x+2, y) if:
> **y + e + m < y + 0.5**

After plotting on each increment of x, we notice that now the error has changed, and how the error changes depends on whether we pick (x+2, y+1) or (x+2, y).
If we pick (x+2, y+1), the new error is now:
> **e_new** = (y + e_old + m) - (y + 1)
> **e_new** = e_old + m - 1

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/brehensam_algo/brehensam_3a.png" alt="brehensam_3a" width="375"/>

__Figure 10: New error for picking (x+2, y+1)__

If we pick (x+2, y), the new error is now:
> **e_new** = (y + e_old + m) - (y)
> **e_new** = e_old + m

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/brehensam_algo/brehensam_3b.png" alt="brehensam_3b" width="375"/>

__Figure 11: New error for picking (x+2, y)__

Take note that the error always takes reference from the y coordinate of the currently choosen grid!

# Pseudocode

With the preceding concepts in place, we are ready to form our pseudocode:
```
# Set error to zero
e = 0 
# Calculate gradient
m = (y2 - y1)/(x2 - x1)

y = y1

FOR x = x1 to x2
    CALL plot(x,y)
    IF ( e + m < 0.5)
        # Increment error by m (error offset taken from y)
        e += m
    ELSE 
        y += 1
        # Increment error by m but error offset taken from y+1
        e += m -1
    END IF
END FOR
```

__Code Block 1: Brehensam's Line Algorithm for Octant 2__

You can try implementing the above in your favourite programming language, but for now let's try implementing this in Golang! 

```
//GOLANG IMPLEMENTATION

m := (float64(y2 - y1)) / (float64(x2 - x1)) //calculate gradient

for x, y, err := x1, y1, 0.0; x != x2+1; x += 1 {
    // Plot point (x,y)
    img.Set(x, y, color)
    if (err + m) < 0.5 {
        // Increment error by m (error offset taken from y)
        err += m
    } else {
        y += 1
        // Increment error by m but error offset taken from y+1
        err += m - 1
    }
}

```
__Code Block 2: Code Block 1 implemented in GOLang__

We will choose to draw a 12 pointed star (Figure 12) to demonstrate that Brehensam is able to handle all signs and magnitude of gradients, by passing in the following input, which will be parsed as x1, y1 -> x2, y2
```
8,8 -> 4,0; 
8,8 -> 0,4; 
8,8 -> 12,0;
8,8 -> 16,4;
8,8 -> 16,12;
8,8 -> 12,16;
8,8 -> 4,16;
8,8 -> 0,12;
8,8 -> 8,0;
8,8 -> 0,8;
8,8 -> 8,16;
8,8 -> 16,8;
```
__Code Block 3: Input given to my programme__

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/brehensam_algo/12_pointed_star.png" alt="12_pointed_star" width="150"/>

__Figure 12: 12 pointed star__

But wait... running our algorithm now will simply put us in an infinite loop where both x and y and incremented, which means that for some lines, (err + m) will always be more than or equal to 0.5 

Turns out our pseudo code only applies for the second octant, which is highlighted in figure 14. Our current algorithm is only able to handle positive gradient values between 1 and 0.

<img src="../public/assets/2021-12-24-battle_of_lines_brehensam/brehensam_algo/octant_highlight_2.png" alt="octant_highlight_2" width="375"/>

__Figure 14: Octant 2 is highlighted here among the other octants__

# What about the other 7 Octants?

## Dealing with gradient values between 1 and INFINITY

Lets solve for the case that we have a positive gradient value between 1 and infinity. There will be a few small changes made to the algorithm:

1. We will need to set the new gradient value as the reciprocal of the actual line gradient.
2. Iterate y from y1 to y2 (Instead of iterating through x)

```
e = 0 
# Calculate reciprocal of gradient
m = (x2 - x1)/(y2 - y1)

x = x1

FOR y = y1 to y2
    CALL plot(x,y)
    IF ( e + m < 0.5)
        e += m
    ELSE 
        x += 1
        e += m -1
    END IF
END FOR
```

__Code Block 4: Brehensam Octant 1__

## Dealing with negativity

We can use the easy trick of swapping the end and start points of the line, to make the gradient negative, but let's try another way around it.

```
#...

FOR x = x1 to x2
    CALL plot(x,y)
    IF ( e + m > -0.5)
        e += m
    ELSE 
        y += 1
        e += m + 1
    END IF
END FOR
```

__Code Block 5: Brehensam Octant 3__

This can similarly be applied to gradient values where -INFINITY < m < 1.

# Getting rid of floating points

So far everything we have talked about involves floating point operations, but that can be the bane of all evil when it comes to efficiency. So naturally we would want to simplify it to only perform integer based operations. 

The following equations will be further massaged to get rid of the floating point variable m:
> **e + m < 0.5** *(1)*
> 
> **e = e + m** *(2)*
> 
> **e = e + m - 1** *(2)*

For **equation (1)**, we will multiply both sides by dx, where dx = x2 - x1:
> **e + m < 0.5** *(1)*
> 
> (e + m) * dx < 0.5 * dx 
> 
> (e + (dy/dx)) * dx < 0.5 * dx 
> 
> 2*e * dx + 2*dy < dx 
> 
> 2*(e' + dy) < dx (_where e' = e*dx_)
> 
> __e * dx + dy < 0.5 * dx__ *(4)*

For **equation (2)**, we also multiply both sides by dx:
> **e = e + m** *(2)*
> 
> e * dx = (e + m) * dx 
> 
> __e' = e' + dy__ *(5)*

For **equation (3)**, the same thing:
> **e = e + m - 1** *(3)*
> 
> __e' = e' + dy - dx__  *(6)*

Now our new pseudocode for octant 2 becomes:
```
e = 0 
y = y1
dy = y2 - y1
dx = x2 - x1

FOR x = x1 to x2
    CALL plot(x,y)
    IF ( 2(e' + dy) < dx)
        # Increment error by m (error offset taken from y)
        e' += dy
    ELSE 
        y += 1
        # Increment error by m but error offset taken from y+1
        e' += dy - dx
    END IF
END FOR
```

### A slight problem for actually drawing images 
However, we have a slight problem.
Conventionally, image coordinate frames have their origin at the top left, with the x direction being positive towards the right, and y direction towards the bottom.
 
We will need to vertically flip the 8 octants for our model to work

## Can I has a house?

<Show the house I drew>

## What about circles? Can I have them too?

Brehensam has been extended to be drawn for circles too

## Anti-aliasing you say? 

However, some of the issues that Brehensam face is the aliasing effect. The lines appear jagged up close and people do get sick of retro-looking graphics after a while. 

Talk about Wu's algorithm and lead up to next article


## Notes
- Diagrams were made with [excalidraw](https://excalidraw.com/)
- I tried to follow [3] for the pseudocode syntax, if you know of any better pseudocode formats please ping me!
- Please let me know if there are any inconsistencies or misstated facts at john_tanguanzhong@hotmail.com

## References
1. [Brehensam's Line Algorithm Pseudocode](https://www.cs.helsinki.fi/group/goa/mallinnus/lines/bresenh.html)
2. [Paul E. Black. Dictionary of Algorithms and Data Structures, NIST.](https://xlinux.nist.gov/dads/HTML/bresenham.html)


[An Introduction to Writing Good Pseudocode](https://towardsdatascience.com/pseudocode-101-an-introduction-to-writing-good-pseudocode-1331cb855be7)




# CHECKS TO DO

1. Figures
   1. Are they numbered properly?
   2. Are they correct?
   3. Are the descriptions filled up?
2. Table of contents
   1. Are the headings done properly?
3. Crediting
   1. Are all sources credited properly?
   2. 