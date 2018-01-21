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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./examples/undistort_road.png "Road Transformed"
[image3]: ./examples/threshold.png "Binary Example"
[image4]: ./examples/warped.png "Warp Example"
[image5]: ./examples/find_and_fit.png "Fit Visual (1)"
[image6]: ./examples/fit_only.png "Fit Visual (2)"
[image7]: ./examples/final.png "Final Result"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup

#### 1. Provide a Writeup that includes all the rubric points and how you addressed each one.   

You're reading it! There is also an IPython notebook "Advanced Lane Finding.ipynb", where all the code including with explanations and example images can be found. All code cell numbers reference to this notebook.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first three code cells of the IPython notebook.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  For some images it was not possible to detect all corners. I applied the distortion correction to one of these images (calibration1.jpg) using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The same distortion correction is used for all test images and the resulting images are saved in "./output_images/1_undistorted_*.png" (see code cell 4). Here is an example of a raw image and the corresponding undistorted image:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In code cell 5 I defined color transform, gradient and threshold functions, which are combined together in code cell 6 into the main threshold function "combined_threshold". I played around with HLS, LUV and Lab color transformation and recognized, that L channel of LUV made a good job for white lines and b channel of Lab made a good job for yellow lines.

There is a function "direction_threshold" for detecting slanted lines, but it needs time-consuming arctan calculation. Therefore I planned to use gradient threshold after perspective transformation, because I thought it would perform best after that transformation. As it turned out, my color threshold were good enough to have an acceptable lane detection, so in the end I didn't use any gradient functions at all.

In code cell 7 the threshold images for all undistorted test images are calculated and resulting images are saved into "./output_images/2_threshold_*.png". Here is an example:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for perspective transform can be seen in code cell 8. The source points have been determined manually like shown in the course video. The destination points have been chosen to build up a rectangle and also to take the same coordinates for the bottom two points:

```python
src = np.float32([[548, 483],  [740, 483], [1053, 690], [261, 690]])
dst = np.float32([[261, 200], [1053, 200], [1053, 690], [261, 690]])
```

This resulted in the following source and destination points:

|  Source   | Destination |
| :-------: | :---------: |
| 548, 483  |  261, 200   |
| 740, 783  |  1053, 200  |
| 1053, 690 |  1053, 690  |
| 261, 690  |  261, 690   |

All thresholded test images are transformed and resulting images are saved into "./output_images/3_warped_*.png". Furthermore one undistorted image has been warped in order to see, if transformation works correctly:

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The functions for finding and fitting lane lines have been taken over from course material. They can be found in code cell 9. To check if they work correctly, one of the test images has been used. Here you can see the output of function "find_and_fit", which is used if no information about the lane line is available (code cell 10):

![alt text][image5]

And here you can see the output of function "fit_only", which is used in subsequent fitting steps (code cell 11):

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature and position of vehicle is calculated in code cell 12. There you can see the two functions "calculate_curvature" and "calculate_offset". The curvature radius is calculated for both left and right lane line separately, but later on I decided to take the mean value of both.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Finally in code cell 13 all test images are processed in terms of curvature radius and vehicle offset calculation. For this the thresholded and warped images are taken and calculation is done. Then lane area is drawn on empty image, inverse perspective transform is done and overlayed with original, undistorted images. At the end curvature radius and vehicle offset are printed on image. All resulting images are saved into "./output_images/4_final_*.png". Here is an example:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_final.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

After finishing the project and thinking about problems and issues, I can't find any big problem related to the project itself. It was clear what to do and all the techniques have been presented in the course material. Sometimes one has to try different approaches, but in the end the final goal and the way how to come there was visible.

What I struggled with were some peripheral issues:

* For each step in pipeline I saved the resulting images and loaded them back in the next step. Dealing with RGB and BGR color ordering was obvious. Also the fact that saving a grayscale image resulted in a three-channel image was manageable. But one pitfall took me a lot of time: saving a grayscale image with Matplotlib's imsave function resulted in a three-channel image with different values for RGB. There is also an [open issue](https://github.com/matplotlib/matplotlib/issues/3657) on this.
* My PC is not the fastest one. Processing the project video took several minutes. Despite the fact, that my pipeline worked good on the test images, there were some glitches on the project video. So I had to process it several times, which resulted in a lot of time. If the PC was faster, I could have done more parameter tweaking or playing around with combination function.

The pipeline worked acceptably on project video, but failed completely on both challenge videos. Obviously color channel threshold (focusing on white and yellow) is not enough for a robust pipeline. Therefore a first improvement would be to analyse the challange videos frame by frame, make use of gradient functions and find a better combination of both color and gradient functions.

As a second step I would try to make line finding algorithm more sophisticated. Currently the lines are treated as independant, but in reality they're not. The distance between both lines could be assumed as a fixed value, also the curvature should be in similar ranges. I would also try to tweak the parameters (number of windows, window size, ...) or re-think the complete algorithm (not just looking on two highest values in histogram, but starting from the middle and looking to the left and right).

Finally the averaging function could be improved. There is a numpy average function, which allows weighted averaging. With it you could give more weight to the more recent fit values and less to the older ones. This may allow to increase the number of fit values for averaging without resulting in too lazy behaviour when the curvature changes.