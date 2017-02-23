# Finding Lane Lines on the Road

The goal of the project is to create an image processing pipeline that would
identify the road lane lines in the images taken by a car frontal camera. As a
result, the pipeline should draw 2 straight lines over the lane lines found in
the image.

This is what the result would look like: 

![Annotated image](./images/report/laneLines_thirdPass.jpg)

### Pipeline description

The pipeline I implemented consists of the following steps.

The source image is converted to grayscale, and the image is slightly
smoothened using the Gaussian filter with the kernel of size 7:
![Grayscale image](./images/report/pipeline_grayscale.jpg)

Next, I apply the Canny algorithm to detect the edges of potential lane
lines, with the following thresholds: 
```python
canny_params = {'low': 120, 'high': 180}`
```
![Canny edges](./images/report/pipeline_edges.jpg)

In order to remove irrelevant details from the image, I apply a region
mask that would only keep the lines in front of the car. The region I'm
interested in is a trapezoid, roughly resembling the shape of the road: 
![Clipped edges](./images/report/pipeline_clipped.jpg)

Using the Hough transform, I detect the potential straight lines in the region
of interest. Empirically, I came up with the following parameters for the Hough
transform that work reasonably well:
```python
hough_params = {'threshold': 30, 
                'min_line_length': 20, 
				'max_line_gap': 30}
```
Using these values, the line segments are identified as follows:
![Identified line segments](./images/report/pipeline_line_segments.jpg)

As a final step, I produce a resulting image, drawing the found line segments
into the original image: 
![Final image](./images/report/pipeline_final_image.jpg)

### Average lane lines

In order to draw the lane lines as two straight lines, rather than a bunch of
line segments, I derive the parameters (slope and intercept) by averaging the
slopes and intercepts of found line segments. 

First of all, I separate the segments with the positive and negative slopes, for
right and left lane lines, respectively. Second, I decided to filter out the
segments that look too vertical or too horizontal, to reduce the
distortion. Finally, the slopes and intercepts for each lane line are calculated
by averaging those of individual segments.
![Final image](./images/report/pipeline_average_lane_lines.jpg)

### Apply the pipeline to video clips

Using the `moviepy` library, it's easy to apply the pipeline to a video
clip. Please see the video clips `lane_solidWhiteRight.mp4` and
`lane_solidYellowLeft.mp4` in the `videos/out` directory. 

### Potential shortcomings

The pipeline, as currently implemented, has the following disadvantages:

1. The parameters for Hough transform are quite tough in order to reduce the
false positives. As a result, they work pretty well on clear pictures, but fail
to find the lane lines when there are interferences, like shadows or reflections
from the road surface. 

2. The pipeline doesn't take into account that there can be other signs drawn on
the road surface, that would interfer with the detection algorithm. 

3. The algorithm would not be reliable when the lanes are significantly curved
(sharp turns).

### Possible improvements

There can be several improvements to the algorithm. For individual images, I
suggest trying to use the color data to emphasize the lane lines. We know that
the lane lines are mostly white or yellow, and we could probably use color
filtering to make lane lines stand out.

Another improvement for individual images would be taking into account the line
segments' lengths when calculating the average. Longer line segments should
probably contribute more to the average than the short ones. 

When applying the algorithm to the video clips, I think we can make use of the
historical data over the sequence of images. Averaging the lane line parameters
over the series of past calculations can help filtering out short distortions,
like occasional shadows, or even additional signs on the road surface.

