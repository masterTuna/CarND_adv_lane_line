**Advanced Lane Finding Project**

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

[image1]: ./output_images/undistort_output.png "Undistorted"
[image2]: ./output_images/warped_image.png "warped chessboard"
[image3]: ./output_images/road_image_undistort.png "test image before/after undistort"
[image4]: ./output_images/color_binary_combine.png "Color/Binary Example"
[image5]: ./output_images/warped_straight_curve_lines.png "Warp Example"
[image6]: ./output_images/inv_warped_curve_line.png "Inv Warp Example"
[image7]: ./output_images/polynomial_fit.png "Fit Visual"
[image8]: ./output_images/inv_warped_curve_line.png "Fit Visual"
[image9]: ./output_images/lane_fill.png "Output"
[video1]: ./out_project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

---
My code is in adv_lane_lines.ipynb.
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

This part of code is in section ”Camera Calibration". For all the test chessboard images, first to convert them to gray image and find the corners in the board. The corners are saved in `imgpoints`, while `objpoints` is only a copy of coordinates. With the data above, apply `cv2.calibrateCamera` to compute the coefficients for distortion. Using `cv2.undistort` to undistort the test images.

![alt text][image1]

I also random picked an image test the warp_image function. Extract its corners and use the the top-left, top-right, bottom-right, bottom-left corner as the source to compute its perspective transform M. After getting M, a warped image could be created by `cv2.warpPerspective`. Here is an example:

![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

First extract distortion matrix from previous checkboard undistortion. Apply the parameters to `cv2.undistort` with test road image. Here is an example:
![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to get the binary image, including sobel, maginitute and direction with different threshold. Functions are defined in section "useful functions". Code to combine the binary and color thresholds are in secton "color/gradient combination". I tried HLS first and found that sometimes it couldn't detect correct spots for the lane due to the shades. So I changed to RGB and tuned the threshold. I am using the following thresholds:

| color/gradient | threshold |
|:--------------:|:---------:|
|sobel           |  30, 60  |
| mag            |  30, 60  |
| dir            |  0.7，1.3 |
| rgb            |  220, 255 |

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

There are two steps to do perspective transform. First, in section "warp road image", my function `warp_line` reads in an image and checks if the image is already binary image. For an binary image input, the image is supposed to be undistorted. Otherwise, an undistortion and conversion to gray image is performed, in line 2 to 7.
With undistorted image, as the lane on the image is straight line, I draw straight lines and choose 4 corners: bottom-left, top-left, top-right, bottom-right, namely:
[260, 681], [592, 452], [686, 452], [1042, 681]. The warped image would be a rectangle, so I choose the bottom left to be w/4 and bottom right to be 3w/4, where w is the width of the image. The height of the rectangle is h. codes in line15 to 19. 

by `cv2.getPerspectiveTransform`, transform M and inverse M could be computed.
The warped image and its inverse warped image are shown here
warped image:
![alt text][image5]
inv-warped image
![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

This part of code is in section "processing image with sliding window".
So I combined the color and binary gradient/amplitude/dir and warped the image to show parallel lanes(line7 to line16). From line 79, I get the x position of the two lane lines in the image histogram where the corresponding y is max. The x for left lane and right lane lie on the two sides of the middle line. Starting with this x position, a sliding window with margin of 100 on both side of cur x position is applied to search the marked pixels. All valid pixels are saved to leftx/lefty and rightx/righty. So from polynomial fit, I can get the coefficients. So the histogram and the polynomial fit are shown here:
![alt text][image7]
![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
In section "calculate curvature", I have a function `get_curvature_in_meter` to calculate the radius in meters. First map pixel space to real distance, I am using:
* ym_per_pix = 40/720
* xm_per_pix = 3.7/700
and in code line12 to line16 the radius can be computed.

Assuming the mirror is in the middle of the car, so the car is located in the middle point of the bottom of the image. Since the left lane and right lane are detected, so the offset is the distance from middle point to the middle of the two lanes. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.
Here is an example clearly shows the lane(in light green)
![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./outproject_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In the video, I found that if the cars run into the shade area or the road has both dark color and light color or the lane is not so clear, the boundary of the color may confuses the code. Since the detected results is significantly different from correct lane lines, code will do blind window search. However the shade part still may be treated as real lane starting point, leading to a failure. By switching to other color space, such as RGB, the code is much more robust. Also I took a look at the challenge video, and the lanes in that video has abrupt turns with very curvy lanes. I'd think 2nd order polyfit may not work well here, maybe higher order polyfit would help.
