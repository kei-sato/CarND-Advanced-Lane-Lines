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

[image1]: ./output_images/undistort_checkboard.jpg "Undistorted"
[image2]: ./output_images/undistort.jpg "Road Transformed"
[image3]: ./output_images/binary_images.jpg "Binary Example"
[image4]: ./output_images/warp.jpg "Warp Example"
[image5]: ./output_images/window_fit1.jpg "Fit Visual"
[image6]: ./output_images/window_fit2.jpg "Fit Visual"
[image7]: ./output_images/curvature_position.jpg "Output"
[image8]: ./output_images/draw_lane.jpg "Output"
[image9]: ./output_images/failure1.gif "Failure"
[image10]: ./output_images/success1.gif "Success"
[video1]: ./output_images/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cell #1,2 of the IPython notebook located in "./main.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The code for this step is contained in the code cell #4 of main.ipynb.

Here is an example of an undistorted image of one of the test images:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this step is contained in the code cell #5,6 of main.ipynb.

I used a combination of S value of HSV color space and the result of the Sobel operation taking the derivative in x.  Here's an example of my output for this step:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for this step is contained in the code cell #7,8,9 of main.ipynb.

First I found out the source points manually (#7,8), and I chose the destination points in the following manner:

```python
pts = np.array([(582, 458), (263, 675), (1047, 675), (699, 458)])
src = np.float32(pts)
offsetx = 300
offsety = 0
x1 = offsetx
y1 = offsety
x2 = img_size[0] - offsetx
y2 = img_size[1] - offsety
dst = np.float32([[x1,y1],[x1,y2],[x2,y2],[x2,y1]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 582, 458      | 300, 0        | 
| 263, 675      | 300, 720      |
| 1047, 675     | 980, 720      |
| 699, 458      | 980, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. (#9)

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this step is contained in the code cell #10,11,12 of main.ipynb.

After perspective transform and binary threshold (#10), to identify lane-line pixels, I took the following aproach (#11):

1. Identify the coordinate of x of the bottom of the lanes by taking a histogram of the bottom half of the image.
2. Using that coodinate as a center, calculate the vertices of the window in which the pixels of the lane-line are supposed to be.
3. Count the number of the white pixels in the window and update the center of the next window if the number was over 50.
4. Back to 2 until the window get to the top of the image.
5. Fit a second order polynomial to the lane-line pixels which are all white pixels in the windows

Here's the results for this step:

![alt text][image5]

As you can see, that doesn't work well in some images. But I'll discuss about the aproach against this problem in the discussion section later.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for this step is contained in the code cell #16 of main.ipynb.

I calculated the radius of curvature at the bottom of the lane line with [this formula](http://www.intmath.com/applications-differentiation/8-radius-curvature.php).  But it's still represented by pixel, so I needed to convert pixels to meters by certain ratios.

And position of the vehicle was figured out by the distance between the center of the lanes and the center of the image.  It's also converted to meters from pixels.

Here's the results for this step:

![alt text][image7]

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for this step is contained in the code cell #14 of main.ipynb.

Here is an example of my result on test images:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The code for this step is contained in the code cell #17 of main.ipynb.

Here's a [link to my video result](./output_images/project_video2.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

##### 1. Fitting didn't work well in some test images

At the step of fitting 2nd polynomial. The first aproach I took didn't work well.

![alt text][image5]

So I fixed the code such like the windows of the left and right lanes keep same distance all the time so that they will be parallel.  Here is the result of that aproach (code cell #12 in main.ipynb). 

![alt text][image6]

This worked well.

#### 2. Catastrophic failures of detecting lane line

There were catastrophic failures in the video clip at the first time like this:

![alt text][image9]

I thought those failures happend because I didn't use the previous fitting result from the previous frame. So I fixed to use the previous fitting result. And here is the result of that.

![alt text][image10]

The final code to create the video clip is in the code cell #17 of main.ipynb
