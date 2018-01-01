# Advanced Lane Line Project

---

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/undistort.jpg "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./output_images/undistort_test1.jpg "Undistorted test1 image"
[image4]: ./output_images/binary_tripple_1.jpg "Binary tripple"
[image5]: ./output_images/warped_1.jpg "Warped image"
[image6]: ./output_images/polyfit.jpg "Polyfit"
[image7]: ./output_images/lane_not_selected.jpg "Lane without highlighting"
[image8]: ./output_images/lane_selected.jpg "Lane with highlighting"
[video1]: ./project_video_out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the **first two code cells** of the IPython notebook located in "advanced_lane_lines_project.ipynb". The third cell is an example of an undistorted image of the road view.

I start by preparing "object points", which will be the $(x, y, z)$ coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the $(x, y)$ plane at $z=0$, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the $(x, y)$ pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I applied the `cv2.undistort()` function with the extracted camera calibration and distortion coefficients to the `"test_images/test1.jpg"` in the **third code cell** of the notebook:

![alt text][image2]

and this is how the undistortedd image looks like (side by side with the original):

![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used the L-channel (in HLS), and the B-channel (in LAB), following the suggestion from one of the reviewers. The thresholds for the L-channel and the B-channel were: (210, 255), and (190, 255), respectively. However, I had to normalize the values of the channels:

```python
def pipeline(...):
...
l_channel = l_channel*(255/np.max(l_channel))
...
b_channel = b_channel*(255/np.max(b_channel))
...

```
otherwise, I wouldn't be able to capture lane lines in shadowed regions.

I also experimented with with the R- and S-channel, but both these channels got "tricked" by the shadows (which led to noisy output), so I ignored them in my final submission.

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in the **fifth code cell**.  The `warp()` function takes as inputs an image (`img`), and the variables necessary for `cv2.undistort()` (`mtx` and `dist`). The source (`SRC`) and destination (`DST`) points are used as `global` variables.  I chose to hardcode the source and destination points in the following manner:

```python
height, width, _ = img.shape

SRC = np.float32([
    (540, 450),
    (740, 450),
    (100, 720),
    (1180, 720),
])

DST = np.float32([
    (300, 0),
    (width-300, 0),
    (300, height),
    (width-300, height),
])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 540, 450      | 300, 0        |
| 100, 720      | 300, 720      |
| 1180, 720     | 980, 720      |
| 740, 450      | 980, 0        |

I verified that my perspective transform was working as expected by drawing the `SRC` and `DST` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image:

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I fit my lane lines with a 2nd order polynomial in the **seventh** and **eight code cells**. The results are as follows:

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculate the radius of curvature and the position of the vehicle with respect to the center in the `process_image()` function, in the **13th code cell**.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Here's an image before highlighting the lane:

![alt text][image7]

and here's the same scene with the lane highlighted:

![alt text][image8]


---


### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_out.mp4)


---


### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

**First,** my strategy was to use as few channels/gradients for selecting the yellow and white lines as possible. That's why I limited myself to the L- and B-channels. By reducing the complexity of the spece of possible combinations, I was hoping to achieve a satisfactory result easier. Alas, this approach does not generalize well to new videos, partly because the reliance on clear, easily identifiable-by-color lines.

**Second,** if I were to invest more time, I would develop a pipeline for exploring the thresholds and combinations of color channels and gradients, so that I wouldn't waste time on running a single scenario, then checking that it's not good enough, then running a second scenario, etc. This took waaay too much time, and what I've learned is that it pays to invest in a general purpose methodology for testing and verifying a batch of scenarios.

**Third,** objects (cars, motorcycles) that interfere with the lane can severely impact the identified lane lines. One solution would be to "smooth out" the lines found in subsequent frames. Currently, for a polynomial fit I'm using a mean of three past identified coefficients. But this can be done better, e.g. with the use of exponential smoothing: `New = gamma*New + (1-gamma)*Old`, where `0 < gamma < 1`.

**Finally,** my solution breaks down for more challenging videos because the region of interest is a fixed trapezoid, and if the road is more curvy and going up and down, it might not be possible to capture the lane lines correctly. One solution to this problem is to consider a wider region, but then I would need a more robust procedure for extracting the binary representation of the image.
