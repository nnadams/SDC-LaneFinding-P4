## **Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # "Image References"

[undist]: ./img/undist.png "Undistorted"
[undist_road]: ./img/undist_road.png "Road Transformed"
[edges]: ./img/edges.png "Edges"
[binary]: ./img/binary.png "Binary"
[polyfit]: ./img/polyfit.png "Fit"
[bird_compare]: ./img/birds_compare.png "Bird"
[final]: ./img/final.png "Final"
[video]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./Lane-Finding.ipynb".

20 different calibration chessboard images are read in, and `cv2.findChessboardCorners` is called to attempt to locate the chessboard corners and finally create a camera matrix. Using this I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![undistored board][undist]

### Pipeline (single images)

All final pipeline code is in the third code cell of the Lane-Finding.ipynb notebook. Code from the lectures was specifically used in calibrating the camera and fitting lines.

#### 1. Provide an example of a distortion-corrected image.

Below is an example of a test image that was undistorted. This distortion correction is can be seen looking at the bottom and corners of the image.
![undistorted road][undist_road]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

First I tried converting the image to grayscale and using Sobel edge detection to get the magnitude and direction of the gradient, and then checked various thresholds to create one binary image, shown below. This method proved too noisy when there were shadows or mixed road colors.

![edge image][edges]

I found that color thresholds were effective enough for this solution. I converted the example images to the HLS, HSV, LAB, LUV and YUV color spaces and displayed each channel (see Jupyter notebook). The saturation from HLS, Blue-Yellow from LAB, and lightness from LUV all looked useful. Those channels seemed to capture either both lanes, yellow lane, or white lanes fairly well. In end the I did not make use of the saturation channel because it could become noisy with shadows.

The below image shows a cyan bounding polygon approximately where the lane perspective transformation is done. The transform actually reveals more curve in the lane than is immediately recognizable from the binary image by going far into the distance.

![color binary][binary]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The perspective transform is done simply with the `cv2.warpPerspective` function. I ended up using the points provided by Udacity. Below is an example of my guess at the points on the left and the Udacity points on the right. Both work about the same in the end. My points appear to result in better looking curves (see next section), but the Udacity points are not only dynamic but result in vision that goes farther down the road, providing a better looking video.

![birds compare][bird_compare]

```python
src = np.float32(
    [[(imshape[0] / 2) - 55, imshape[1] / 2 + 100],
    [((imshape[0] / 6) - 10), imshape[1]],
    [(imshape[0] * 5 / 6) + 60, imshape[1]],
    [(imshape[0] / 2 + 55), imshape[1] / 2 + 100]]
)
dst = np.float32(
    [[(imshape[0] / 4), 0],
    [(imshape[0] / 4), imshape[1]],
    [(imshape[0] * 3 / 4), imshape[1]],
    [(imshape[0] * 3 / 4), 0]]
)
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 585, 460      | 320, 0        |
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Fitting second order polynomials to these binary, transformed images started with creating a histogram of number of pixels per X value. With two lanes, two peaks always appeared in the histogram. Those peaks dictated the start of each curve. From there nine windows were used to idenify pixels likely to be a part of the lane lines. Each window was recentered to follow the center of the points. Once all the points were found, a second over polynomial was fit to them. 

![polyfit][polyfit]

#### 5. Describe how you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature, or the average of the both radii of the lanes, was calculated by converting all points into real world coordinates, polyfitting again and calculating the radius for each line. A breakdown of how the formula works can be seen [here][https://www.math24.net/curvature-radius/].

To get the cars distance from the lane center, I assumed the camera is at the center of the car, and thus the center of the image. From the first couple real world points were used to calculate the center of the curve x intercepts, which was compared to the center of the image.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![final image][final]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

There were no major issues, but as discussed above there was a lot of time spent of deciding how to identify the lanes between edge detection, color spaces and how to transform. There is no single method to do this, and a better method would likely combine everything that I tried.

For example, this pipeline does not work well with varied levels of light. Using dynamic thresholds would help that by changing the thresholds based on the perceived brightness of the road. You cannot avoid glare on the lens so it would useful to not just find the lanes, but predict where they go, since you have some idea the curve. This would losing the lanes and having nothing to go one. Another issue is that the view is very static. This is fine on a only slightly curved highway, but on a slower road, with sharp turns, the lanes may curve outside of the search frame or even outside of the image. Losing one lane could be handled by keeping track of how wide the lane is.