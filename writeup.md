# Finding Lane Lines for Self Driving Cars
---


## The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: output_images\camera_cal.png "camera_cal"
[image2]: ./output_images\undistorted_ex.png "undistorted_ex"
[image3]: ./output_images\pres1.png "Prespective1"
[image4]: ./output_images\pres2.png "Prespective2"
[image5]: ./output_images\color1.png "color1"
[image6]: ./output_images\color2.png "color2"
[image7]: ./output_images\color_good1.png "color_good"
[image8]: ./output_images\color_good2.png "color_good"
[image9]: ./output_images\sobel.png "sobel"
[image10]: ./output_images\canny.png "canny"
[image11]: ./output_images\color_pipeline.png "pipeline"
[image12]: ./output_images\fit1.png "fit1"
[image13]: ./output_images\fit2.png "fit2"
[image14]: ./output_images\fit3.png "fit3"
[image15]: ./output_images\final1.png "final1"
[image16]: ./output_images\final2.png "final2"
[image17]: ./output_images\final3.png "final3"

[video1]: ./project_video_output.mp4 "Video"


### Camera Calibration  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

## Pipeline (single images)

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

### Perspective Transform

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image3]
![alt text][image4]


### Color Selection

I Pulled out the individual channels from each color space, and see if it better isolates the laneline pixels.
![alt text][image5]
![alt text][image6]

Clearly some of these channels perform better than others. Let's just look at the good ones.
![alt text][image7]
![alt text][image8]

### Sobel

I defined a function that applies Sobel x and y, then computes the magnitude of the gradient and applies a threshold
![alt text][image9]

### Canny

I also tested canny edge detector
![alt text][image10]

### Binary Pipeline Function

I defined a function that will process an image and return the binary image, and tested this function on all test images
![alt text][image11]

### Finding and Fitting Line Pixels

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:
![alt text][image12]

Using the full algorithm from before and starting fresh on every frame may seem inefficient, as the lines don't necessarily move a lot from frame to frame.

In the next frame of video you don't need to do a blind search again, but instead you can just search in a margin around the previous line position, like in the above image. The green shaded area shows where we searched for the lines this time. So, once you know where the lines are in one frame of video, you can do a highly targeted search for them in the next frame.
![alt text][image13]
![alt text][image14]

### Measuring Curvature

I calculated the radius of curvature for the lane lines based on pixel values, so the radius I'm reporting is in pixel space, which is not the same as real world space. So I repeated this calculation after converting x and y values to real world space (meters). Then, I projected the measurements back down onto the road with the curvature radius and the vehicle offeset from centre of the lane printed into it.
![alt text][image15]
![alt text][image16]
![alt text][image17]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)