
# **Advanced Lane Finding Project**

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

[image1]: ./myoutput/original_image_of_chess.jpg "Undidstorted"
[image2]: ./myoutput/undistorted_image_of_chess.jpg "Undidstorted"

[image3]: ./myoutput/undistorted_image_of_road.jpg "Road Transformed"
[image4]: ./myoutput/S_binary_Image.jpg "Road Transformed"
[image5]: ./myoutput/L_binary_Image.jpg "Road Transformed"
[image6]: ./myoutput/B_binary_Image.jpg "Road Transformed"
[image7]: ./myoutput/binary_warped.jpg "Road Transformed"
[image8]: ./myoutput/26.png "Road Transformed"
[image9]: ./myoutput/warped_image.jpg "Road Transformed"
[image10]: ./myoutput/25.png "Road Transformed"
[image11]: ./myoutput/lane_area_filled.jpg "Road Transformed"


---
## Camera Calibration

### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third and fourth code cell of the jupyter notebook `project4_finalcode.ipynb`

I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
#### Original Image
![alt text][image1]
#### Undistorted  Image
![alt text][image2]

## Pipeline (single images)

### 1. Provide an example of a distortion-corrected image.

#### Undistorted Image of the road
![alt text][image3]
### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image in 9 th code cell in `project4_finalcode.ipynb` . Here are the channels of each color space I used with their min and max threshold.


#### S (of HLS color space) binary Image  (min_threshold=170, max_threshold=255)
![][image4]
#### L (of HLS color space) binary Image  (min_threshold=215, max_threshold=255)
![][image5]
#### B (of LAB color space) binary Image (min_threshold=155, max_threshold=200)
![][image6]
#### Combined binary Image
![][image7]




### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective`, which appears in fourth code cell.  The `perspective()` function takes as inputs an image (`image`) and uses `opencv` function `cv2.warpPerspective`. I have hardcoded the region  which has to be warped 
by selecting four points as `src` which define the region
```
src = np.float32([[550,470],[790, 460],
                      [160, 720],[1250, 720]])
dst = np.float32([[0, 0], [1280, 0], 
                     [0, 720],[1280, 720]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 550, 470      | 0, 0        | 
| 160, 720      | 0, 720      |
| 1250, 720     | 1280, 720      |
| 790, 460      | 1280, 0        |

#### Points selected for warping(starred)
![alt text][image8]
#### Warped Image
![alt text][image9]

### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I have computed a histogram and found peaks in left and right parts of image and formed a rectangle window around those peaks and then stack those windows which follow the line by checking and if the number of points in each window is greater than 50 then change the x position of the window to the mean x position of the points. I maintain  record of all the points in the window and then used polyfilt() functions to fit  a curve to those points.

Once I fit a curve in a frame of video, for the next frame I donot repeat the whole procedure but search within the region around the pervious fitted curve

![alt text][image10]

### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius by the following code which takes care of the conversion from pixels to meters which is implemented in lines 40 through 206 of code cell 16 in my code in `project4_finalcode.ipynb`
```
ym_per_pix = 30./720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meteres per pixel in x dimension
left_fit_cr = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)
right_fit_cr = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)
left_curverad = ((1 + (2*left_fit_cr[0]*np.max(lefty) + left_fit_cr[1])**2)**1.5) \
                             /np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*np.max(lefty) + right_fit_cr[1])**2)**1.5) \
                                /np.absolute(2*right_fit_cr[0])
```
I found out the midpoint of the left and right curves intersect at the bottom of the image and also found midpoint of the image and the if first midpoint is greater than the second midpoint vehicle is to the right of the center of the lane else to the left of the center of the lane

### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in a function process_vid which is in lines 187 through 206 of code cell 16 in my code in `project4_finalcode.ipynb`.  Here is an example of my result on a test image:

![alt text][image11]

---

## Pipeline (video)

### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://www.youtube.com/watch?v=dcxi2QQVW00&feature=youtu.be)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
The pipeline is fairly robust to varying light conditions. It also did a decent job on other videos except in shadow where it momentarily lost
I feel there should be a better way to find out the thresholded binary image than finding it by trying different color channels with different thresholds 

