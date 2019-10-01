# Self-Driving Car Engineer Nanodegree Program
## Advanced Lane Finding Project
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

## The goals and steps to develope a pipeline for recognizing of the road in front of the automous car
* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/calibration18.png
[image2]: ./output_images/undist_image.png
[image3]: ./output_images/color_only_combined_img.png
[image4]: ./output_images/pers_trans_image.jpg
[image5]: ./output_images/lane_find1.png
[image6]: ./output_images/lane_find2.png
[image7]: ./output_images/lane_find4_21.jpg
[image8]: ./output_images/lane_find3.jpg

#### 1.Provide a  Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I am using the OpenCV functions `findChessboardCorner` and `calibrateCamera` as the backbone of the image calibration. I use these functions because the camera who to took the photos from the chessboard images on the wall is the same as the camera in the car. So we have few pictures from the chessboard image from different angels to do the calibration. First of all, i fed the function `calibrateCamera` with arrays of object points, corresponding to the location of internal corners of a chessboard image, and image points, the pixel locations of the internal chessboard corners that i determined by `findChessboardCorners`. This will retrun me the camera calibration and distorition coefficients. I will use these to undisort the images with the OpenCV function `undistort` because all pictures are generally distortet taking into account the camera lense. The below image depicts the corners drawn onto twenty chessboard images using the OpenCV function `drawCessboardCorners`:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The effect of `undistort` is minimal and can be seen at the hood of the car:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used 4 different thresholds and combined them into one. You can see in the jupyter notebook where i sued the different thesholds and combined them, clearly. I used the `binary_threshold`, `abs_sobel_threshold`, `color_R_G` and `color_L_S` as it is seen in the following image:  

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I fist use the `combined threshold` on the pictures, after that i `undistored` and use the `perspective transformation`. I also marked the region that the perspective is transformed to. The functions are named after there functions in jupyter notebook.+

This is the `source` and the `destination` for the `perspective transformation` function:

```
src:
    top_right = [700,458]
    bottom_right = [1040,686]
    bottom_left = [250,686]
    top_left = [580,458]
dst:
    top_right = [1040,0]
    bottom_right = [1040,686]
    bottom_left = [250,686]
    top_left = [250,0]
```
This is the picture after using the `comibined thresholds`, `undistortion` and `perspective transformation`:

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The functions `fit_polynomial`(Slinding Window Search) and `search_around_poly`(Search From Prior), which identify lane lines and fit a second order polynomial to both right and left lane lines, are clearly labeled in the Jupyter notebook as "Sliding Window Polyfit" and "Polyfit Using Fit from Previous Frame". The first of these computes a histogram of the bottom half of the image and finds the bottom-most x position (or "base") of the left and right lane lines. Originally these locations were identified from the local maxima of the left and right halves of the histogram, but in my final implementation I changed these to quarters of the histogram just left and right of the midpoint. This helped to reject lines from adjacent lanes. The function then identifies ten windows from which to identify lane pixels, each one centered on the midpoint of the pixels from the window below. This effectively "follows" the lane lines up to the top of the binary image, and speeds processing by only searching for activated pixels over a small portion of the image. Pixels belonging to each lane line are identified and the Numpy `polyfit()` method fits a second order polynomial to each set of pixels. The image below demonstrates how this process works:

**Slinding Window Poylfit**
![alt text][image5]

**Search from Prior**
![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is based upon [this website](http://www.intmath.com/applications-differentiation/8-radius-curvature.php) and calculated in the code cell titled "Radius of Curvature and Distance from Lane Center Calculation" using this line of code (altered for clarity):
```
left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])
  
average_curved_rad = (left_curverad+right_curvedrad)/2
```

The position of the vehicle with respect to the center of the lane is calculated with the following lines of code:

```python
car_position = img_size[1]/2
l_fit_x_int = left_fit[0]*y_eval**2 + left_fit[1]*y_eval + left_fit[2]
r_fit_x_int = right_fit[0]*y_eval**2 + right_fit[1]*y_eval + right_fit[2]
lane_center_position = (r_fit_x_int + l_fit_x_int) /2
center_offset_mtrs = (car_position - lane_center_position) * xm_per_pix
```

The direction of the curvature is computed:
```
if center > 0:
        direction = 'right'
elif center < 0:
    direction = 'left'
```

![alt text][image7]

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![alt text][image8]

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

[![Video of my pipline](https://img.youtube.com/vi/9S9xiFzlOiY/0.jpg)](https://youtu.be/9S9xiFzlOiY)

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
The problems I encountered were almost exclusively due to lighting conditions, shadows, discoloration, etc. This porblem can be seen in the video i linked above, clearly. At some point the street can not be marked and that would kill the pipline. For this problem i used the class `Line`. With the class i could save good lanes and use it if the road could no be seen from my pipeline.
