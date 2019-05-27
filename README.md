# **Finding Lane Lines on the Road** 

## Writeup

This is a simplistic approach in detecting road lane lines. My personal goal was to familiarize myelf with the technologies and craft up a solution that:
1. is easy to read and understand
2. maintains good code quality
3. performs excellent on the first 2 videos 
4. performs adequately on the challenge video

In this repo you will also find frame-by-frame exports of the test videos, exported to disk using `clip.to_images_sequence("images%03d.jpeg")`. Usefull if you need to debug specific frames.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on the work in a written report


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
It extends from the bottom of the screen up to where the lanes tend to meet. We call this spot the `horizon`.

# Find Hough lines
![alt text][hugh_lines]

We detect line segments out of the edges detected in the previous steps. This step also calls `draw_lines()` which unifies all line segments into a single line for each road lane. 

In order to draw a single line on the left and right lanes, `draw_lines()` was modified to best fit a straight line to the end points of the line segments that were detected out of each lane. As a first step we ignore all the line segments which have too high or low slope. Line segments that tend to be horizontal or vertical probably are not lanes and we are interested in them. Then, the `polyfit` function is used to find the coefficients of `y = mx + b` of the best-fit line. Additionally `poly1d()` is used to easily calculate y values out of x. Using that we extrapolate the lines from the bottom of the screen up to the horizon which is roughly at the vertical center of the image.

The final result looks like that:

![alt text][overlayed]

### 2. Identify potential shortcomings with your current pipeline

Some shortcomings noticed:
- One shortcoming is that in some frames, parts of the image between the two lanes are detected as edges, which affects the polyfit results.
- In the challenge video the car hood hides a portion of the lanes and tracking seems to be more fragile that the first 2 videos.
- It would also be interesting to see how the pipeline performs at night or with direct sunlight.


### 3. Suggest possible improvements to your pipeline

Possible improvements would be:
- Use `polyfit()` to calculate x out of y for the lane lines (instead of y out of x). This will help us with extrapolation as we will only need to the "horizon" y-level to caclulate the x value of each extrapolated line.
- For parameter tuning it would be helpful to extract all video frames with overlays after we run our pipeline. This way it would be easy to find out which frames are the problematic ones and run our pipeline on those frames individually in order to tune our parameters.
- We could do automatic car hood detection as part of our pipeline to crop that part of the image automatically if the car hood is visible.
- Define a more complex region of interest that ignores the part of the image between the two lanes.