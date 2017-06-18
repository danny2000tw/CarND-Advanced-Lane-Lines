## Advanced Lane Finding

---

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

[image1]: ./examples/undistort_output2.png "Undistorted"
[image2]: ./examples/GradientX.png "GradientX.png"
[image3]: ./examples/HSV_thresholding.png "HSV_thresholding"
[image4]: ./examples/combined_thresholding.png "combined_thresholding"
[image5]: ./examples/perspectiveTransfrom.png "perspectiveTransfrom"
[image6]: ./examples/histogram.png "histogram"
[image7]: ./examples/final_image.png "final_image"


### 1. Camera Calibration

Camera doesn't create perfect images, some of the objects in the images especially ones near the edges can get stretched or skewed in various ways, and we need to correct for that. The following are the steps of how I undistort the camera images so that we can get correct and useful information out of them.

* Use multiple chessboard images to calibrate the camera
* Create a transform that maps these distorted points (2d points in image plan) to undistorted points (3d points in real world space)
* Use openCV `findChessboardCorners` to detect the chessboard corners in the 2d images
* Finally, to calibrate the camera, use openCV `cv2.calibrateCamera()`, the API will return the distortion coefficients and the camera matrix that we need to transform 3d object points to 2d image points via `cv2.undistort` API.

![alt text][image1]


### 2. Binary Threshold Image
Next, I used a combination of color and gradient thresholds with a region of interest mask to generate a binary image that identifies the lane lines (logic defines in sub method `binary_thresholds_pipeline`). With lane finding, we know ahead of time that the lines we are looking tend to be close to vertical. By taking  advantage of the fact, we applied image with sobel_x operator, which is a way taking the derivative of the image in the x direction, to pick up edges closer to vertical. 

![alt text][image2]

Further more, I also take advantage of the information from color spaces to make our line detecion more robust. Thresholding on *Saturation* channel in HLS color space helps up still do a fairly good job of picking up the lines under different lighting and contrast conditions.

![alt text][image3]

The following is the result of combining two methods with equation:
`color_grad_binary[(grad_x == 1) | (s_binary == 1) ] = 1`

![alt text][image4]

### 3. Perspective Transformation (Bird Eye View)

Then, I perform a perspective transform to get a birds eye view of the lane. This allows us to fit a polynomial to the land line which we couldn't do very easily before, and to find the curvature of the lines so that autonomous car can predict the best steering angles. To compute the perspective transform matrix, we need to provide the source and destination points to openCV `cv2.warpPerspective` API. The following is the src and dst coordinates that is used for the transformation.

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 695, 460      | 960, 0        | 
| 1120, 720     | 960, 720      |
| 300, 720      | 320, 720      |
| 565, 460      | 250, 0        |

![alt text][image5]

### 4. Dectect Lane Lines 

Now we have a thresholded warped image (birds eye view), we can then plot the histogram on the x axis and use the highest peak as a starting point to perform sliding windows approach to detect the potential lane lines. Apply `np.polyfit` with the points discovered by the sliding windows to find the second order polynomial to each left and right lane line.

![alt text][image6]

### 5. Calculate Curvature Radius and Offcenter

Please refer to the `Helper Functions` section in `Advanced-Lane-Lines.ipynb` to see how I calculate the curvature radius and the offset of the vehicle to the center.

### 6. Apply pipeline on a single test image 

![alt text][image7]
### Pipeline (video)

Here's a [link to my video result](https://youtu.be/HmUnX1csC8Q)
### Discussion

Currently, the pipeline is not robust enough to detect lane lines with shadows, or under different lighting and contrast conditions. Challenge here is to define an appropriate threshold that works well with all these conditions. A dynamic changing thresholding mechanism might be a good approach to go to make the system more robust based on different scenarios. Another improvement I can think of is that instead of detecting the land line per frame, we could average the result of say 10 - 15 frames to rule out the outliers because of the sudden lighting change or shadows. 