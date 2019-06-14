## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


In this project, your goal is to write a software pipeline to identify the lane boundaries in a video, but the main output or product we want you to create is a detailed writeup of the project.  Check out the [writeup template](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) for this project and use it as a starting point for creating your own writeup.  

Creating a great writeup:
---
A great writeup should include the rubric points as well as your description of how you addressed each point.  You should include a detailed description of the code used in each step (with line-number references and code snippets where necessary), and links to other supporting documents or external references.  You should include images in your writeup to demonstrate how your code works with examples.  

All that said, please be concise!  We're not looking for you to write a book here, just a brief description of how you passed each rubric point, and references to the relevant code :). 

You're not required to use markdown for your writeup.  If you use another method please just submit a pdf of your writeup.

The Project
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

[calibration]: ./camera_cal/calibration2.png
[cam_calibration]: ./output_images/camera_cal.png
[undistort_calibration]: ./output_images/cam_cal_undistorted.png
[test1_out]: ./output_images/test1_out.png
[threshold_bin_1_out]: ./output_images/threshold_bin_1_out.png
[perspective_trans1_out.jpg]: ./output_images/perspective_trans1_out.png
[sliding1]: ./output_images/sliding1.png
[curvature1]: ./output_images/curvature1.png
[warped]: ./output_images/warped.png
[warped_rad]: ./output_images/warped_rad.png

### Camera Calibration

#### I have explained all concepts with one example image. More images are included in ./output_images folder.

The code for this step is contained in the fourth code cell of the IPython notebook located in "P2.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

#### Original Image

![alt text][calibration]

####  Image Calibration

![alt text][cam_calibration]

#### Undistorted Image

![alt text][undistort_calibration]

### Pipeline for all testing images

#### 1. Undistorted Images

By using camera calibration and distortion coefficients I undistort the image. Undistorted output:

![alt text][test1_out]

#### 2. Color transforms and gradients methods are used to get thresholded binary image

Detecting edges around trees or cars is okay because these lines can be mostly filtered out by applying a mask to the image and essentially cropping out the area outside of the lane lines. It's most important that we reliably detect different colors of lane lines under varying degrees of daylight and shadow. So, that our self driving car does not become blind in extreme daylight hours or under the shadow of a tree.
I performed gradient threshold and color threshold individually and then created a binary combination of these two images to map out where either the color or gradient thresholds were met called the combined_binary in the code.
Following is binary image:

![alt text][threshold_bin_1_out]

#### 3. Perspective Transform

Perspective Transform is the Bird's eye view for Lane images. We want to look at the lanes from the top and have a clear picture about their curves. Implementing Perspective Transform was the most interesting one for me. I used values of src and dst as shown below: 

src = np.float32([[590,450],[687,450],[1100,720],[200,720]])

dst = np.float32([[300,0],[900,0],[900,720],[300,720]])

Also, made a function warper(img, src, dst) which takes in the Binary Warped Image and return the perspective transform using cv2.getPerspectiveTransform(src, dst) and cv2.warpPerspective(img, M, img_size, flags=cv2.INTER_NEAREST). The results are shown below:


##### Prespective Transformation on Thresholded Binary Image

![alt text][perspective_trans1_out]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Once I got the Perspective Transform of the binary warped images, I first used the sliding window method to plot the lane lines and fitted a polynomial using fit_polynomial(img) function. Later on, I used the Search from prior technique and fitted a more accurate polynomial through my perspective transformed images using search_around_poly(image) funtion. Proper markings are there in the code to indicate each and every step.

##### Finding the lines - Sliding Window and fitting a polynomial

![alt text][sliding1]

##### Finding the lines - Search From Prior and fitting a polynomial

![alt text][curvature1]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

For calculating the radius of curvature and the position of the vehicle with respect to center, I made a function called radius_and_offset(warped_image) which returns curvature_string and offset. Used left_lane_inds and right_lane_inds for performing the task. Used function fit_poly(image.shape, leftx, lefty, rightx, righty) which returns left_fitx, right_fitx, ploty to calcuate the real radius of curvature and offset.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

After implementing all the steps, it's time to create the pipeline for one image. Created a function process_image() as the main pipeline function. Also, I put the Radius of Curvature and Center Offset on the final image using cv2.putText() function. The result is shown below: 

##### Lane Area Drawn without Information:

![warped][warped] 

##### Lane Area Drawn with Radius of Curvature and Central Offset:

![warped_rad][warped_rad]

---

### Pipeline (video)

The code for this step is contained in the IPython notebook named "P2.ipynb".

Here's a [link to my project output video result](./project_video_output.mp4)
Here's a [link to my challenge output video result](./challenge_video_output.mp4)
Here's a [link to my hard challenge video result](./harder_challenge_video_output.mp4)

---

### Discussion

My pipeline will fail when the road curves a lot and there is a sudden change in the direction like the roads in hill areas as the case in hard-challenge-video. I need to improve my pipeline such that it can detect all type of curves correctly.
