# **Finding Lane Lines on the Road** 

## Writeup

This is a simplistic approach in detecting road lane lines. My personal goal was to familiarize myelf with the technologies and craft up a solution that:
1. is easy to read and understand
2. maintains good code quality
3. performs well

In this repo you will also find frame-by-frame exports of the test videos, exported to disk using `clip.to_images_sequence("images%03d.jpeg")`. Usefull if you need to debug specific frames.
---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[grayscale]: ./examples/grayscale.png "Grayscale"
[blurred_image]: ./examples/blurred.png "Blurred image"
[canny_edges]: ./examples/canny_edges.png "Canny edges"
[masked_region]: ./examples/masked_region.png "Masked region"
[hugh_lines]: ./examples/hugh_lines_extrapolated.png "Hugh lines"
[overlayed]: ./examples/final.png "Final"


---

### Reflection

### 1. Description of the pipeline

The jupyter file is unchanched except for the `draw_lines()` and the `process_image()` functions. This makes it easier for people familiar with the problem to follow this solution. I made use of all the helper functions.

My pipeline consists of 5 steps:

# Convert to grayscale
![alt text][grayscale]

The is a simple grayscale conversion.

# Blur
![alt text][blurred_image]

We blur the image before the next step in order to soften our sharp edges.

# Perform Canny edge detection
![alt text][canny_edges]

This will detect the edges in the image. Our goal is to detect only lane lines and not any other edges. 

# Define a region of interest
![alt text][masked_region]

We define our region of interest as a polygon with 4 vertices. It has the shape of a 2D frustum being wide in the bottom and narrow in the top. It basically follows the lanes shape.
It nomrally extends from the bottom of the screen up to where the lanes would meet. We call this spot the `horizon`. In our case we also crop a bit in the bottom to compensate for the car hood that is visible in the challenge video.

# Find Hough lines
![alt text][hugh_lines]

We detect line segments out of the edges detected in the previous steps. This step also calls `draw_lines()` which unifies all line segments into a single line for each road lane. 

In order to draw a single line on the left and right lanes, `draw_lines()` was modified to best fit a straight line to the end points of the line segments that were detected out of each lane. For that `polyfit` was used which returns us the coefficients of `y = mx + b`. In addition `poly1d()` is used to easily calculate y values out of x. Using that we extrapolate the lines from the bottom of the screen up to the horizon which is roughly at the vertical center of the image.

The final result looks like that:

![alt text][overlayed]

### 2. Identify potential shortcomings with your current pipeline

One potential shortcoming would be what would happen when the is horizontal edge detected coming from a shadow, tree branch etc. It would also be interesting to see how the pipeline performs at night or with direct sunlight.


### 3. Suggest possible improvements to your pipeline

Possible improvements would be:
- Use `polyfit()` to calculate x out of y for the lane lines (instead of y out of x). This will help us with extrapolation as we will only need to the "horizon" y-level to caclulate the x value of each extrapolated line.
- For parameter tuning it would be helpful to extract all video frames with overlays after we run our pipeline. This way it would be easy to find out which frames are the problematic ones and run our pipeline on those frames individually in order to tune our parameters.
- On cases such as shadows on the road it can be than lane lines can be detected on those shadows. Sometime this shadows are almost vertical. We can see such a case in the last challenge video. We could ingore edges that are have an almost vertical slope. This will also help ignoring items that have fallen on the street, such as tree branches or trash.
- We could do automatic car hood detection as part of our pipeline to ignore that part of the image automatically if the car hood is visible.