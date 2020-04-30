## Advanced Lane Finding Project Writeup


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


[video1]: ./project_video.mp4 "Video"
[image_calib1]: ./camera_cal/calibration1.jpg "Calibration Image 1"
[image_calib1_undist]: ./output_images/calibration1_undistorted.jpg "Calibration Image 1 Undistorted"
[image_straight_line1]: ./test_images/straight_lines1.jpg "Original test image straight line 1"
[image_straight_line1_undistorted]: ./output_images/distortion_corrected/straight_lines1.jpg "Test image straight line 1
 after distortion corrected"
[image_stln1_binary_thresholded]: ./output_images/thresholded_binary/straight_lines1.jpg "Binary thresholded image of 
 test image straight line 1 "
[image_stln1_top_view]: ./output_images/top_view/straight_lines1.jpg "Top bird eye view of straight line 1 after 
perspective tranform"
[image_stln1_fitted_lane]: ./output_images/fitted_lane/straight_lines1.jpg "Fitted lane on top view of straight line 1."
[image_stln1_overlay_original_annotated]: ./output_images/overlay_original_annotated/straight_lines1.jpg "Straight line 1
 with lane area drawn and radius of curvature and vehicle offset drawn"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in `./my_solution.ipynb`.

For this step, the chessboard images in camera_cal folder were read and the findChessboardCorners was used to find the 
inner corners in the chessboard pattern. Since we know that each image has 9 by 6 corners, we use this information to 
calculate the camera calibration matrix mtx and distortion coefficient dst.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. 
Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each
calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a 
copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with
the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using 
the `cv2.calibrateCamera()` function.  

I applied this distortion correction to a calibration image  ![alt text][image_calib1] using the `cv2.undistort()` 
function and obtained this result:
![alt text][image_calib1_undist]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

After obtaining the camera matrix and distortion coefficients from the camera calibration step, each of the images in 
test_images folder is undistorted using cv2 undistort function. The resulting undistorted images are put into 
`distortion_corrected` sub folder of `output_images` folder, with the same filename as the original file name.

One of the original image ![alt text][image_straight_line1] and its corresponding undistorted image is
 ![alt text][image_straight_line1_undistorted]. 

Much difference cannot be seen between the original image and its undistorted image in most of the area except on the 
lower portion of the image containing the hood of the car.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of gradient and saturation thresholds, then applied masking for region of interest, to generate a 
binary image.  The code for this step is contained in the third code cell of the IPython notebook located in
 `./my_solution.ipynb`.

A thresholded binary image is formed from the undistorted image using the following process:

* An image is obtained after applying threshold on the sobel x gradient.
    * the undistorted image is converted to gray scale
    * sobel x gradient is obtained from the gray scale image and then normalized to 0 to 255 value using the maximum 
    value of soble x gradient.
    * the normalized sobel x gradient is then compared with the supplied minimum and maximum threshold for sobel x 
    gradient.
* Another image is obtained by applying threshold on the saturation channel.
    * the undistorted image is converted to HLS image and S channel is obtained.
    * threshold for S channel is applied to get the thresholded image
* Both of the thresholded images are combined using OR logic.
* Only the image portion in the region of interest is obtained. Since we are interested in finding the lanes border of
 the single lane on which the car is currently driving and we need to remove the noise in unwanted areas, a trapeziod
  (nearly triangular) type of region of interest in-front of the hood of the car is used.
* Some noise filter is done by closing and then opening the pixels using 5 by 5 rectangular matrix

For each of the undistorted images, the thresholded binary image is obtained and saved into thresholded_binary subfolder
 of output_images folder. 

For undistorted image  ![alt text][image_straight_line1_undistorted], its thresholded binary images is also shown below:
![alt text][image_stln1_binary_thresholded]



#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

In this step, first we find the perspective matrix using a reference image and expected position in top view image and
 calling method getPerspectiveTransform with those corresponding positions as source and destination points. Using the 
 perspective matrix, the binary thresholded images are converted to top view and saved into top_view sub folder of
  output_images folder.   

The code for this step is contained in the fourth code cell of the IPython notebook located in `./my_solution.ipynb`.

Here, undistorted image of straight_lines2.jpg is taken as reference. Taking nearest dashed line, dot and the next 
dashed line as reference for the left side, and comparing to Detail 12 of 
https://dot.ca.gov/-/media/dot-media/programs/traffic-operations/documents/ca-mutcd/camutcd2014-part3-rev3-a11y.pdf , 
the length of the section is 60ft and the width of the lane is standard 12 ft.
 
 Casting this lane section of 60ft by 12 ft to the whole image, such that the length of the lane section is equal to 
 the height of the image, it resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 565,475       | 568,0         | 
| 265,700       | 568,720       |
| 722,475       | 712,0         |
| 1050,700      | 712,720       |

The reference image straight_lines1.jpg can be found in google map at 
 <https://www.google.com/maps/@37.440167,-122.2488739,3a,75y,298.54h,93.8t/data=!3m6!1e1!3m4!1sZ2Tgn_cip1A02fCGc190Fg!2e0!7i16384!8i8192 >. 
There also the ratio of the dashed line, dot and the next dashed line, to the width of the road, is around 5. Hence the
 assumption made should be nearly accurate. Also the ratio of the length to the width of the road in the bird view image
 is also around 5 for the referenced image.

For a sample thresholded binary images  ![alt text][image_stln1_binary_thresholded], its top bird eye view image after the 
perspective transformation is shown below:
![alt text][image_stln1_top_view]



#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In this step, the pixels of the right and left lane boundary are determined on the topview image and then a second order
 polynomial line is fit on those detected pixel for each side of the lane boundary. The code for this step is contained 
 in the fifth code cell of the IPython notebook located in `./my_solution.ipynb`.

The detection of the pixels belonging to the left and the right lane boundary are done in the method find_lane_pixels, 
where a sliding window approach has been taken to detect each of the lane boundary. The histogram of the lower half 
image in the top view image is taken, in order to determine the initial x position of the left and right lane borders. 
Then position of the sliding window is changed to the new mean position if the number of detected pixels is greater than
 minimum required pixels (here 50 pixels).

In the method fit_polynomial, the pixels detected for left and right lane border using method `find_lane_pixels`, are used
 to fit a second degree polynomial line for each side lane border, that approximates the position of the detected lane 
 borders. Form the fitted polynomial, the pixels belonging to the lane border are determined and plotted. 
 
 In some case, where there are not enough pixels detected along the length of the lane border, there fitted line for 
 given y position from 0 to height of the image, had the x positions greater than that of the width of the image, in 
 such lane lines pixels are not plotted.

For the top view image of straight_lines1.jpg ![alt text][image_stln1_top_view],  its corresponding detected lane pixels,
 sliding window and calculated pixels from fitted lines is shown below:
 ![alt text][image_stln1_fitted_lane]
 

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for this step is contained in the sixth code cell of the IPython notebook located in `./my_solution.ipynb`.
 
In the `get_fit_polynomial_parameters` method, the lane pixels belonging to the left and right lane borders are found 
using the method `find_lane_pixels`. Then using the length per pixels in x and y directions, the second order polynomial 
is fitted on the pixels points (converted to real distances) for each side of the lane borders.

In method `measure_curvature_real`, the left and the right fitted line are obtained from method `get_fit_polynomial_parameters`. 
The radius of the fitted left and right line are calculated for the bottom position of the image. Similarly the x 
position of the left and right lane are calculated at the bottom of the image and the average is taken as the center 
position of the lane. The mid point on the bottom horizontal line of the image is taken as the position of the car. The 
difference between the car position and the lane center is then taken as the distance of the car from the mid of the 
lane. While displaying the curvature of the radius of the car, the average of the radius of the curvature of the left 
and right lane border can be taken.



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The main code for this step to identify lane area on the original undistorted image is contained in the seventh code 
cell of the IPython notebook located in `./my_solution.ipynb`. The code to annotate the radius of curvature of the lane 
and the offset of the vehicle position is in the eight code cell of the IPython notebook.

In this step, the detected lane boundaries are indicated by overlaying a light green color over the detected lane area 
on the undistorted original image.

In the method `get_fitted_polynomial_line`, the right and left lane pixels are obtained using method `find_lane_pixels`.
Then left and right polynomial line are fit over the detected right and left lane pixels, and then use the fitted 
polynomial to get a smooth lines representing left and right lane.

Then in the method `get_overlaid_image`, the pixels of the lines representing left and right lane are used in the OpenCV 
method `fillPoly`, to draw the lane onto a blank image. Then the blank image (with lane drawn by method `fillPoly`) is 
wrapped back to original image space using inverse perspective matrix (Minv). Afterwards, the warped image is then 
combined with the undistorted picture.

For an original test image ![alt text][image_straight_line1], the annotated image with the lane area, radius of lane 
curvature and vehicle position from lane center is shown below:
![alt text][image_stln1_overlay_original_annotated] 

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

In order to process the video, a method process_pipeline has been created combining the previous steps explained earlier.
The code for this step is contained in the ninth code cell of the IPython notebook located in `./my_solution.ipynb`.
 
The following steps are performed on the original image passed to the method:
* get the undistorted image first from original image using `cal_undistort` method.
* get undistorted thresholded image after thresholding the undistorted image using method `get_thresholded_binary_image`.
* get the top view of the binary threholded image from the undistorted thresholded image using method 
`get_perspective_transformed_image` .
* get the lane boundary overlaid image from the binary thresholded image using method `get_overlaid_image`.
* get the curve and position from the binary thresholded image using `measure_curvature_real` method.
* annotate the radius and distance on the lane boundary overlaid image using `draw_data` method.
* Finally this annotated image is returned by the method `process_pipeline`.

This `process_pipeline` is passed to the `fl_image` method as image function in order to process the project video 
file `project_video.mp4` . The output video is saved as `project_video_output.mp4` file.

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The main problem faced during the project was while obtaining the thresholded binary image. There were many options to
choose from. I tried to use color thresholding for yellow and white lines. I also tried to smooth the L channel of HLS
image using CLAHE algorithm. But most of them failed, may it needed more tuning. Finally, I settled on using thresholds 
in S channel of HLS image and thresholds in Sobel x gradient for this project, after a lot of unsuccessful attempts 
using other combinations. Even then I see some glitches and wobbly lines in some cases.
 
The pipeline could fail in cases where there are difficulty in identifying lane borders, which could be due to less 
amount of lane border in the camera image, or due to sharp turns (as I am assuming lane line are in-front of the camera 
 only) or due to some shadow or change in lane color on some extreme cases. 

May be if I had more time, I would try using various combinations to determine the binary thresholded image. But given 
that I have already spent a lot of time for some number of combinations/trials, may be I need some guidance/mentoring 
from some experienced person, if I have to improve on it. I could of course try reading some more related research paper
and try applying.
