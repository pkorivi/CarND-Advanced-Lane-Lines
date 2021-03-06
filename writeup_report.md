## Writeup - Advanced Lane Finding

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

[image1]: ./out_images/calib_original.png "Calibration"

[image3]: ./out_images/lane_binary_1.png "Binary Example"

[image4]: ./out_images/transformed_perspective.png "Perspective Transformed Imaged"

[image5]: ./out_images/lane_identified.png "Fit Visual"
[image6]: ./out_images/lane_final.png "Output"
[video1]: ./out_project_video.mp4 "Video"

### Steps in Detail

### Camera Calibration

#### 1. Camera Calibration Steps.

The code for this step is contained in the Camera Calibration code cell of the IPython notebook [lane_finding.ipynb](./lane_finding.ipynb).

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection from multiple images in multiple angles for the images. 

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1] 

### Pipeline (single images)

#### 1. Distortion-correction for images.

Here is an example of the distortion corrected image:
![alt text][image2]

#### 2. Thresholding of images.

I used a combination of color and gradient thresholds to generate a binary image.The code for this step is contained in function 'get_thresholded_img()' of the Helper functions code cell of the IPython notebook [lane_finding.ipynb](./lane_finding.ipynb).  Here's an example of my output for this step.

![alt text][image3]

#### 3. Perspective transformation for image.

The code to calculate the perspective transform and inverse transform matrix is in the code cell 'Perspective transform Matrix' of  the IPython notebook [lane_finding.ipynb](./lane_finding.ipynb). The code to transform the images is in the function 'get_perspective_transformed_image()' in the Helper functions code cell

The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
y_max = int(undistorted.shape[0]*0.93)
y_min = int(undistorted.shape[0]/2+100)

x_l_b = int(undistorted.shape[1]*0.21)
x_r_b = int(undistorted.shape[1]*0.82)
x_r_t = int(undistorted.shape[1]*0.55)
x_l_t = int(undistorted.shape[1]*0.45)

src = np.float32([[(x_l_b,y_max),(x_r_b,y_max), (x_r_t, y_min),(x_l_t,y_min)]])
dst = np.float32([[(x_l_b+50,y_max),(x_r_b-50,y_max), (x_r_b-50, 1),(x_l_b+50,1)]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 270, 670      | 320, 670      | 
| 1050, 670     | 1000,670      |
| 700, 470      | 1000,1        |
| 580, 470      | 320, 1        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Identifying lane lines and fit a polynomial

The code to identify the lane line pixels and fitting a polynomial to it is in the code cell 'Helper functions to find the lines' of the IPython notebook [lane_finding.ipynb](./lane_finding.ipynb). The functions 'find_initial_fit()' and 'find_next_fit()' find the lanes, the next_fit function takes in input the previous lane to keep the margin og search close to the found lane. 


![alt text][image5]

#### 5.Calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code to find radius of curvature and distance to center is in the code cell 'Helper functions to find the lines' of the IPython notebook [lane_finding.ipynb](./lane_finding.ipynb). The functions 'find_initial_fit()' and 'find_next_fit()' also calculate the radius of curvature and distance to center in the end of the function marked '#Radius of curvature, #distance to center'. 

#### 6. Final image creation with lane marking.

The code to create the final image with lane markings is in the function 'get_final_image()' of the code cell 'Helper functions to find the lines' of the IPython notebook [lane_finding.ipynb](./lane_finding.ipynb). 

![alt text][image6]

---

### Pipeline (video)

#### 1. Final: Processing of video.

The function to process the images from final video is in code cell 'Pipeline to process the images from video' of the IPython notebook [lane_finding.ipynb](./lane_finding.ipynb)

Here's a [link to my video result](./out_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

There were many problems in solving this challenge:
- The first is identifying the right ways to threshold images, I tried multiple color spaces and multiple thresholing ways but everything failed to some extant. I finally ended up using the initial model discussed in the course which isn't performing the best. 

- The next problem is using the lane parameters to provide better fit to the lanes in case of wrong detections. I have implemented a simple memory of using previous good lane if there are bad fits but this didnt go well for the other challenge videos. Only the project video was working good. 

