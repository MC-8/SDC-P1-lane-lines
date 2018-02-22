# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

[image11]: ./test_images_output/1_solidYellowLeft.jpg "Blurred"
[image2]: ./test_images_output/2_solidYellowLeft.jpg "Grayscale"
[image3]: ./test_images_output/3_solidYellowLeft.jpg "Edge detected"
[image4]: ./test_images_output/4_solidYellowLeft.jpg "Edge + Mask"
[image5]: ./test_images_output/5_solidYellowLeft.jpg "Hough segments"
[image6]: ./test_images_output/6_solidYellowLeft.jpg "Hough segment on image"
[image7]: ./test_images_output/7_solidYellowLeft.jpg "Averaged/extrapolated Hough lines"
---

### Reflection

My pipeline consisted of the following steps. First, I converted the images to grayscale, then I applied a gaussian blur on it. This is to improve the results of the Canny edge detection algorithm.
![alt text][image1]
lol
![alt text][image2] 


Once a Canny edge detection algorithm was applied to the image, 
![alt text][image3] 
I masked the result with a region of interest that would include just the two lane lines up to the horizon. 
![alt text][image4]
The result of this algorithm returns a series of pixels corresponding to the areas with high contrast change (highest gradient changes) that can be visually interpreted as lines. Our algorithm does not know at this point about lines, it just knows about pixels.
The next step is to obtain lines from sequence of pixels, trying to focus only on very long sequences of pixels that are more likely to be the one composing the lane lines.
To do that, the Hough transform is applied to this data, to obtain end-points of segments, candidates to be our lines of interest.
![alt text][image5]
![alt text][image6]
At this point many lines may have been generated, and it is desired to display only two lines: one for each line limiting the driving lane.
To obtain one line per each side, one way to do it is to average the lines, but first there need to be a way of knowing which lines belong to which side.
I chose to categorize the lines by their slope. Left-Right lines in this project have clearly positive or negative slopw depending on their position, so in this case it works ok. It also has the advantage to ignore near-horizontal lines especially present in the challenge video (car bonnet), and the second video (sporadic horizontal lines).
![alt text][image7]
At this point we have one line per side, but the result from the video (especially the challenge one), only mildly satisfactory: even if the lines are correctly displayed on top of the lane lines, the slight visual disturbance causes them to flicker or change position/orientation altogether. Lines are further filtered out based on their intercept with y-axis.
To improve the result, a final step is applied, where the left-right lines are "remembered" and updated at every frame, taking into consideration the previous position of each line. The idea is that consecutive video frames are not likely to introduce significant changes, so the new information is only used to correct, and not replace, the old information. This is also known as applying a low-pass filter.

The approach to discriminate lines based on their slope approach may not work during sharp turns, but sharp turns probably may need to use a non-linear interpolation.

Another shortcoming could be that the lines averaged are not part of the same "group", and outliers can significantly skew away the line.

The "line filtering" currently uses 20% of new information. This may not be enough to track rapid changes in the road, or if the frame-rate is low.

For efficiency, it may be worth to mask the image, before applying the edge detection.
Also, lines in output of the Hough Transform should be further filered and grouped together. One way to group together lines is to apply an extra Hough Transform on the line obtained from the previous result, and consider cluster of many points together (representing lines with similar slope and intercept, hence close to each other).

