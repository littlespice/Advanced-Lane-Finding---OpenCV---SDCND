

------

# Advanced Lane Finding Project - Jeongil Ju

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



##### [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points are covered in the sections below

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  

File included:

- This writeup document and an HTML copy

- P4-Advanced-Lane-Finding.ipynb

- Output videos : challenge_result.mp4, harder_challeng_result.mp4

  

## Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

There are 20 calibration images are provided by Udacity and saved in ./camera_cal/ folder. These images are used to compute the camera matrix and compute distortion coefficient. These calibration parameter will then be used to undistort the road images taken with the same camera.

First off, we need to prepare 9 x 6 x 3 arrays to store 3-dimensional coordinates for real world object points of the chessboard corners, since there are 9 x 6 check board corners. 

For 20 calibration images, `cv2.findChessboardCorners()` will find 9x6 corners and save those coordinates to `obj_points` and  `img_points`. These corner data from 20 calibration images will then be used to compute calibration paratemters (namely mtx, dist, rvecs, tvecs) from `cv2.calibrateCamera()`. Here is a sample of a calibration image with detected 9 by 6 chessboard corners.

![alt text][image1]



The `mtx` and `dist`  from cv2.calibrateCamera() are then used in `cv2.undistort()` to correct distortions in the images that are taken by the same camera. Here are some results of undistorting function.

![alt text][image2]



## Pipeline (test images)

#### 1. Provide an example of a distortion-corrected image.

Here's a sample of road images and undistorted outputs, using the same undistort parameters from the previously mentioned results of calibration images.


![alt text][image3]

#### 2. Describe how you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

For gradient thresholds, there are 3 examples presented in the classroom: `abs_sobel_thresh()`, `mag_thresh()` and `dir_threshold()`. I used sobel threshold respect to x axis combined with directional thresholds to eliminate unnecessary horizontal lines. 

- Sobel threshold to x-axis : Min(10), Max(200)
- Directional threshold : Min(pi/4), Max(pi/2)
- Maginitude treshold : Not used

For color thresholds, I combined both R and G channel threshold outputs of RGB colorspace with S and L channel threshold of HLS colorspace. This approach gives a good estimation of white and yellow colored lane markings.

- R and G channel thresholds in RGB colorspace : Min(150), Max(255)
- L channel threshold in HLS colorspace : Min (120), Max(255)
- S channel threshold in HLS colorspace : Min (100), Max(255)

The above mentioned threshold outcomes are then combined into an output image. Then, this output image is then roi(region of interest) mask, leaving only bottom half of the image and taking out unnecessary information such as sky and trees. 

Here are some examples of binary output results found in the code block [7]. 

![alt text][image5]

![alt text][image6]

#### 3. Describe how you performed a perspective transform and provide an example of a transformed image.

In order to transform the binary output image containing possible lanes line marking information to a top-down view image, I used warp() with hand-coded transform matrix. Ultimately, you want to transform a trapezoid drawn along the two lane lines of the original road image (source coordinates) to a rectangle of the new top-down images (destination coordinates.

```python
# Source points to be warped (after many tries...)  
src = np.float32(
    [[width/2 - 70, height/2 + 110],
    [width/2 - 420, height],
    [width/2 + 470, height],
    [width/2 + 82, height/2 + 110]])

# Destination points to be warped   
dst = np.float32(
    [[width/4, 1],
    [width/4, height],
    [width/4+600, height],
    [width/4+600, 1]])
```

These sets of coordinates are then used in `cv2.getPerspectiveTransfrom()` and `cv2.warpPerspective()` to warp the input image to a top-down view and reverse. The source coordinates are critical in correctly transforming the image for lane detection. You can see the trapezoid is successfully transformed to a rectangle shape aligning with straight lane markings as described in code block [8]

![alt text][image7]

#### 4. Describe how you identified lane-line pixels and fit their positions with a polynomial?

From the histogram of warped binary output at the bottom quarter of the image, you can find the left and right lane locations in x-axis. From this bases, you can run sliding windows to get fit indicators for lane marking following up to y-axis direction. With these fit lane indicators, the left and right lanes are finally identified using the following polynomial equations respectively shown in code block [11] and [12.

- Left lane : `left_fit[0]*ploty**2 + left_fit[1]*ploty + left_fit[2]`
- Right lane : `right_fit[0]*ploty**2 + right_fit[1]*ploty + right_fit[2]`

Finally, these identified positions are fit with second order poly nomials using `np.polyfit()`.

![alt text][image8]

![alt text][image9]

#### 5. Describe how you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Code for calculating the radius of curvature was already introduced in the classroom. I used np.polyfit() to determine the curvature and used the equations introduced [here](https://en.wikipedia.org/wiki/Radius_of_curvature) to calculate the radius. As shown in the ipynb, it is 30 meters per pixel in y and 3.7 meters per pixel in the original image. 

After the warping to a new perspective and judging by the warped images, these numbers become:

- Meters per pixel in x = 3.7 / 600

  (3.7 meters between two straight lanes) / (600 pixels width in warped image)

- Meters per pixel in y = 15 / 720

  (Approximately 5 dotted lines where each is 3 meter) / (720 pixels high )

So this gives, Calcuting an offset from the center of the lane ( left plus right lane locations divided by 2) to the image center (supposedly the camera is mounted at the center of the vehicle) will give you the relative position to the center. The code is shown in code block [19].

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Here is an example of my result on a test image. The code and outputs are described in code block [17].

![alt text][image10]

---

## Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

![alt text][video1]

---

## Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I tried the same pipeline with the harder challenge video, and my model often failed where roads are significantly covered by tree shadows. There could be several counter measures to this problem.

- 2 lane lines are always separated by a fixed amount of distance from legislation or traffic laws. Alwlays limiting the distance between 2 detected polymonial lines will likey to improve woobly lines to appear.
- Directional thresholds are applied to the pipeline and they set between pi/4 (45 degrees) and pi/2 (90 degrees). Refering these directional threshold from the previoulsy detected lane lines will probably improve performance both curved lanes and straight lanes, because there won't be a sudden changes of lane line shapes from straight to curves within a few frames.       



[//]: #	"Image References"
[image1]: ./images/chessboard.png
[image2]: ./images/undistorted_list.png
[image3]: ./images/undistorted_road.png
[image4]: ./images/binary_output0.png
[image5]: ./images/binary_output1.png
[image6]: ./images/binary_output2.png
[image7]: ./images/testresult.png
[image8]: ./images/poly1.png
[Image9]: ./images/poly2.png
[image10]: ./images/final_output.png
[video1]: ./challenge_result.mp4

