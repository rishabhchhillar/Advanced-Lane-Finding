# Advanced Lane Finding Project

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

[im01]: ./output_images/original-chessboard.png "Original Chessboard"
[im02]: ./output_images/undistorted-chessboard.png "Undistorted Chessboard"
[im03]: ./output_images/original-highway.png "Original Highway"
[im04]: ./output_images/undistorted-highway.png "Undistorted Highway"
[im05]: ./output_images/threshold-gradient.png "Threshold Gradient"
[im06]: ./output_images/threshold-magnitudepng "Threshold Magnitude"
[im07]: ./output_images/threshold-gradient-direction.png "Threshold Gradient Direction"
[im08]: ./output_images/combined-thresholds.png "Combined Magnitude and Direction Thresholds"
[im09]: ./output_images/color-channels.png "Color Channels"
[im10]: ./output_images/thresholded-s.png "Thresholded S Channel"
[im11]: ./output_images/combined-threshold.png "Combnined Color and Gradient Thresholds"
[im12]: ./output_images/warped.png "Warped Highway"
[im13]: ./output_images/thresholded-s-warped.png "Thresholded S Channel Warped Highway"
[im14]: ./output_images/sliding-windows.png "Sliding Windows"
[im15]: ./output_images/average-lines.png "Similar Lines from Average"
[im16]: ./output_images/detected-lane.png "Detected Lane"
[im17]: ./output_images/detected-lane-metrics.png "Detected Lane with Metrics"
[im18]: ./output_images/processed-image.png "Processed Image"

[video1]: ./project_video_output.mp4 "Output Video"

## Rubric Points

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

This step is performed using the camera_calibration method, in which I first defined the chessboard size and prepared the object and image points. Then I loaded the chessboard images, converted them to grayscale, and used the findChessboardCorners from OpenCV to detect the corners. The output from that function was then used in calibrateCamera function to perform camera calibration.
To undistort the images I used the undistort function from OpenCV.

![alt text][im01]
![alt text][im02]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The image below depicts the results of applying the `undistort` function from cv2 to a test image of the highway.

![alt text][im03]
![alt text][im04]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I explored several gradient thresholds to perform binarization. Below are the results of some:

![alt text][im05]
![alt text][im06]
![alt text][im07]
![alt text][im08]

I also played around with some color thresholds for performing color transforms:

![alt text][im09]
![alt text][im10]

In my final pipeline, I decided to combine the gradient and gradient direction thresholds to detect edges. For the color thresholds, I chose to use the R, G, S channels to detect the lanes, and the L channel to avoid pixels which have shadows.

Below is the result of the combined color and gradient transforms:

![alt text][im11]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The Perspective Transform is performed using the `warp` function, which takes source and destination points and computes the perspective transform and the inverse perspective transform using the `getPerspectiveTransform` function from cv2.

![alt text][im12]
![alt text][im13]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The complex process of detecting lane line pixels has been done using several functions. First, I used the Sliding Windows technique in the function `find_lane_pixels`, in which I set the number and margin of windows and stepped through the windows one by one to identify the nonzero pixels, append them to the left_lane_inds and right_lane_inds lists, and recenter the next window on their mean position if found > minpix pixels.
I then extracted the left and right line pixel positions and used them to fit a second order polynomial to each of them using `np.polyfit`.

![alt text][im14]

I also computed the average of 12 previous frames using the Search from Prior method in the `average_lines` and `similar_lines` functions in order to make up for undetected lane lines in some frames.

![alt text][im15]

The points for left and right lines detected from these functions were then used to draw the detected lane onto the warped image using the `draw_lane` function.

![alt text][im16]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The curvature of the lanes is calculated using the `curvature` function, in which I first fit a second order polynomial to pixel positions in each fake lane line, and then fit new polynomials to x and y in real world space using the predefined conversions in x and y from pixel space to meters.

To calculate the offset of the car from the center, I first took the horizontal mid-point of the image, then the position of the car w.r.t. the lane, and then finally calculated the offset by multiplying their difference by the conversion of x from pixels to real world space. This has been done in the `car_offset` function.

![alt text][im17]

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I combined all the above mentioned functions in the `Pipeline` class, which performs all the necessary functions and gives out a complete processed image as the result as such:

![alt text][im18]

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's the final [output video](./project_video_output.mp4).

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

##### Issues faced:

* In some of the frames no lane lines were detected or were very poorly detected, especially in areas where the road had a patch.
* It was also difficult to detect lanes properly where the lighting varied greatly and frequently.

##### Solutions:

* To solve this issue, the sliding windows search was performed again in order to find a better average of the previous frames.
* Different binarizations and sanity checks were also tried out for better detection.
