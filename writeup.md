## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./output_images/stage0/Undistort.jpg "Undistorted"
[image2]: ./output_images/stage0/Undistort_test5.jpg "Road Transformed"
[image3]: ./output_images/stage1/binary.jpg "Binary Example"
[image4]: ./output_images/stage1/birdseye.jpg "Warp Example"
[image5]: ./output_images/stage1/roi.jpg "Fit Visual"
[image6]: ./output_images/stage1/project_test5.jpg "Output"
[video1]: ./processed_project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./stage0_camera_calibration.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at function `binarize` in `stage1_test_image_pipeline.ipynb`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in lines 1 through 15 in the file `stage1_test_image_pipeline.ipynb`  (or, for example, in the 7rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [((img_size[0] / 6) - 23), img_size[1]],
    [[(img_size[0] / 2) - 47, img_size[1] / 2 + 94],
    [(img_size[0] / 2 + 54), img_size[1] / 2 + 94]],
    [(img_size[0] * 5 / 6) + 20, img_size[1]])

new_top_left=np.array([src[0,0],0])
new_top_right=np.array([src[3,0],0])
offset=[150,0]
    
dst = np.float32([src[0]+offset, new_top_left+offset, new_top_right-offset, src[3]-offset])

```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 190, 720      | 340, 720      |
| 589, 460      | 340, 0        | 
| 698, 460      | 995, 0        |
| 1145, 720     | 995, 720      |


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this: The function `find_peaks(img,thresh)` in cell of `stage1_test_images_pipeline.ipynb` takes the bottom half of a binarized and warped lane image to compute a histogram of detected pixel values. The result is smoothened using a gaussia filter and peaks are subsequently detected using. The function returns the x values of the peaks larger than `thresh` as well as the smoothened curve. 

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in function `get_next_window(img,center_point,width)` in my code in `stage1_test_images_pipeline.ipynb`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `stage1_test_image_pipeline.ipynb` in the function `project_lane_lines()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. 
Getting the pipeline to be robust against shadows and at the same time capable of detecting yellow lane lines on white ground was the greates difficulty. I took the approach that lines should never have very low saturation values, i.e. be black. Setting a minimal value for the saturation helped when paired with the x gradient and absolute gradient threshold. Problematic is also the case when more than two lanes get detected. For lines detected far away a threshold on the distance eliminated the problem. By far the most work was implementing the logic of a continuous update of detected lines as well as restarting when the buffer of previous lines emptied. At the moment the pipeline will likely fail in the challenge video. This could be solved by building separate robusted lane line detectors for yellow, white and blur lines together.
