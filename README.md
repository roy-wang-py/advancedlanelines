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

[image1]: ./output_images/undistort_output.jpg "Undistorted"
[image2]: ./output_images/undist_test4.jpg "Road Transformed"
[image3]: ./output_images/binary_combo_test4.jpg "Binary Example"
[image4]: ./output_images/warped_straight_lines.jpg "Warp Example"
[image5]: ./output_images/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/test4_output.jpg "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./P2.ipynb" .  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Undistorted][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Road Transformed][image2]
I have got mts & dist in Camera Calibration, so i use  cv2.undistort to get a distortion-corrected image. 

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.



I tried different COLOR space, and find that LAB's channel B could pick yellow lines, and HSV'v & RGB could pick white lines. Then try sobel_x, dir_gradient, mag_gradient, sobel_x & mag_gradient work well, but still too may other lines picked.
I create 3 pipelines:
    1)pipeline: using sobel_x, HLS
    2)pipeline2:using HLS, LAB, RGB
    3)pipeline3： using HSV, LAB, RGB(similar with pipeline2)
Then test 3 pipelines with test images, and find pipeline3 is a better.

Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![Binary Example][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp_img()`, which appears in lines 1 through 8 in cell "Apply a perspective transform to rectify binary image ("birds-eye view")." (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
h,w = image.shape[:2]

src = np.float32(
        [
            (727,476),
            (1050,689),
            (272,689),
            (560,476)
        ])
dst = np.float32(
        [
            (w-460,0),
            (w-460,h),
            (460,h),
            (460,0)
        ])

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 560, 476      | 460, 0        | 
| 272, 689      | 460, h      |
| 1050, 689     | w-460, h      |
| 727, 476      | w-460, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

During my perspective transform, i try to cut images by choose dst in order to drop useless pixels.

![Warp Example][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used find_lane_pixels to find lane lines in a perspective transform images, and used search_around_poly to search lane lines in a continous image based on pre one.

These two func would find lane lines' points(pixcels), then I used fit_poly to fit these points.

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![Fit Visual][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this by func measure_curvature_and_offset.
To calculate curvature, used:
    left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
    
To get center posion:
    I got left line based point (l,height), and right line based point(r,height), by setting y = height(height is the images' shape in axis y).
    Now i know lane lines' center = (r-l) / 2. Position of the vehicle with respect to center is offset of lane lines' center and images' center.
  

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I used func draw_lane_lines to draw result back to original images.

Here is an example of my result on a test image:

![Output][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

1) My pipeline did not work well with challenge video & hard challenge video.
2) When some other white lines in lane detected, my pipeline would take it as lane lines. Maybe i could make the sliding window size smaller.
3) I have tried used relative value as threshold by values / max(values), while it did not work well in some special scene. 
4) When dealing with challenge video, i update Class Line's fit.While i should check current fit, and drop those had great difference compared to best fit. I tried ,but still work bad. I should improved it.
5) As light differs, using RGB to pick white lines is not reliable.  My pipeline is good at pick yellow line(LAB works well), so I should calculate  white lines  by yellow lines picked.
