## KH Project 4 final writeup

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


## Final Output

Here is the final result of this project, below I describe how these results were obtained

![alt text](writeup_images/ezgif.com-video-to-gif.gif)

[//]: # (Image References)

[imageA]: ./writeup_images/1.PNG "Code"
[imageB]: ./writeup_images/2.PNG "Code"
[image2B]: ./writeup_images/2B.PNG "Code"
[imageC]: ./writeup_images/3.PNG "Code"
[imageD]: ./writeup_images/4.PNG "Code"
[imageE]: ./writeup_images/5.PNG "Code"
[imageF]: ./writeup_images/6.PNG "Code"
[imageG]: ./writeup_images/7.PNG "Code"
[imageH]: ./writeup_images/8.PNG "Code"
[imageI]: ./writeup_images/9.PNG "Code"
[imageJ]: ./writeup_images/10.PNG "Code"
[imageK]: ./writeup_images/11.PNG "Code"
[imageL]: ./writeup_images/12.PNG "Code"
[imageM]: ./writeup_images/13.PNG "Code"
[imageN]: ./writeup_images/14.PNG "Code"
[imageO]: ./writeup_images/15.PNG "Code"
[imageP]: ./writeup_images/16.PNG "Code"
[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code in this step is contained in cell 2 of the ipython notebook, copied below:

![alt text][imageA]

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

![alt text][imageB]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 



### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

The distortion correction uses the following camera calibration and distortion correction matrix found by the previous camera calibration step

![alt text][image2B]

To undistort the image we use the CV2 function below

```
cv2.undistort(img, mtx, dist, None, mtx)
```


The final result of distortion correction is: 
![alt text][imageC]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

A number of color space transformations, gradients, and tresholds were used in the final pipeline. 

![alt text][imageD]
![alt text][imageE]


Colorspace changes from BGR to RGB, to HSV, to grayscal, were all used to pick out particlar features of the image.
Gradients were also used to help isolate lines tending in particular directions. Below we see the sobel operator. 
![alt text][imageF]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The perspective transform makes use of the following cv2 function:

```
    M = cv2.getPerspectiveTransform(src, dst)
    Minv = cv2.getPerspectiveTransform(dst, src)
    # use cv2.warpPerspective() to warp your image to a top-down view
    warped = cv2.warpPerspective(img, M, (w,h), flags=cv2.INTER_LINEAR)
```

Where the source and destination points were defined as follows:

```python
    # define source and destination points for transform
    src = np.float32([(575,464),
                      (707,464), 
                      (258,682), 
                      (1049,682)])
    dst = np.float32([(450,0),
                      (w-450,0),
                      (450,h),
                      (w-450,h)])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 575, 464      | 450, 0        | 
| 707, 464      | w-450, 0      |
| 258, 682     | 450, h      |
| 1049, 682      | w-450, h        |

![alt text][imageG]

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][imageH]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
To detect lane line, eac h image was ran through a pipleline of multiple thresholding. 

The final pipeline is a combination of HSV, HSL, RGB, and Solbel direction and magnitute tresholding until reasonable lane
lines could be detected. The below shows the dectection of the lane lines. The code is showsn in CELL 21 of the Iphython notebook. 

![alt text][imageI]

I used the sliding window approach to detect lane lines. This method uses the histogram of the image as an initial 
starting point to determine where the most probable lines could be. Below is a histogram of the image, coupled with the 
sliding windows

![alt text][imageJ]

A second order polynomial is used to fit to the curve as shown below:
![alt text][imageK]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The below code shows how I calculate the radius of curvature, with an assumed x and y meter per pixel. 

![alt text][imageL]



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The image is overlayed with the poly lines and text showing the curvature and distance from center, then the image is 
warped backed from a topDownView to the orignal view using the inverse tranformation matrix MinV calculated
during the initial warp transformation. 

![alt text][imageM]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline struggled a bit with shadows as was common from the previous projects, although not as bad as bedfore. 
The tresholding parameters were found via hand tuning. I am curious is an approach that makes uses of different an
automated approch like using lease squares error to minimize a function to yield the best thresholded values. 

But I believe having more thresholdings for unique conditions will help yield better results, however this was 
notopted due to the limited resources of my personal computer and the amount of compute time required for each operation 
to be performed. Overall the results were very impressive for a few lines of python code. 


## Update After Mentor Insight

The mentor provided very useful information with regards to using the L channel of HLS and the B channel of LAB to detect the yellow and white lines. A new pipeline was added based on the recommendations to help correct errors with detecting lines in heavy shadows.

In addition, if a lane is not detected, the previous fir line is used. The new pipeline is shown below, and the results shows that the entire video lane is detected very accurately. 

![alt text][imageN]
![alt text][imageO]
