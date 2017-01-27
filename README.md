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

[image1]: ./output_images/calibration.png "Calibration"
[image2]: ./output_images/test4_original.png "Original"
[image3]: ./output_images/test4_undistorted.png "Undistorted"
[image4]: ./output_images/test4_binary.png "Binary"
[image5]: ./output_images/test4_transformed.png "Transformed"
[image6]: ./output_images/test4_line_pixels.png "Line pixels"
[image7]: ./output_images/test4_fit_lines.png "Fit lines"
[image8]: ./output_images/test4_final.png "Final"
[video1]: ./output_images/project_video_output.mp4 "Video"

## Code
All code is contained in the Jupyter notebook "Advanced lane lines.ipynb". I found the notebook especially useful for a project like this, where I could quickly review inline plots, a big part of this project.

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

Udacity provided a number of chessboard images. I detected the chessboard corners using the cv2.findChessboardCorners() function, then used the cv2.calibrateCamera() function. The destination points were the same for all of the images, based on a chessboard directly facing the camera, a simple grid with perpendicular lines.

After calibrating the camera, I applied this distortion correction to the test image using the cv2.undistort() function. Two before and after calibration images are shown below:

![alt text][image1]

###Pipeline (single images)

As an overview of my code, I have two global Line objects that store the state of the left and right lines. The video processing function fl_image() calls my process_wrapper() function, which is a wrapper around my pipeline that returns only the final image used for the output video. process_wrapper() calls process_movie_frame(), which returns all of the intermediate states of the pipeline too, and is used to produce the images below.

For the examples below I use the 'test4' image. Here is the original image:
![alt text][image2]

####1. Provide an example of a distortion-corrected image.
The distortion correction is again done using cv2.undistort() in my image_pipeline() function, using the calibration information that was obtained initially. The resulting image is:
![alt text][image3]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I spent probably the longest amount of time considering various techniques in isolation and combination to achieve a decent binary representation containing the line pixels. Some of the techniques that I ended up not using in the end were:
- Thresholding the red color channel: I compared to thresholding for white and yellow colors separately and found that was more effective.
- X and Y sobel operators, magnitude of gradient, direction of gradient on grayscaled images: I tried various thresholds and combining these but it proved less effective than applying them in a different color space.

Instead, I ended up using two techniques:
- X and Y sobel operators on saturation channel: I transformed the image to HLS and apply tresholds on saturation. This picked up the edges around lane lines effectively.
- Yellow and white thresholds: I transformed the image to HSV, which made it easy to pick out yellow and white thresholds by looking at the Wikipedia diagram of the HSV colorsplace.

Pixels from both of these techniques were combined into a binary image, example shown here:

![alt text][image4]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

My transformation is done in the function transform_perspective(). I played around with a few points before settling on effective source and destination points. I make use of the OpenCV functions cv2.getPerspectiveTransform and cv2.warpPerspective to get the transformation matrix and apply it, respectively.

An example of the transformed image is shown here:

![alt text][image5]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To find pixels that belong to lane lines, I used the approach suggested in the lecture material. If you sum over the columns of the binary image, the peak values are the probably lane line centers. You can sum up columns for the lower portion of the image, then slide this window up the image and in each layer, select points with the same technique. 

To make this approach a bit smarter, rather than serching over the entire width of the image to find the peak value, the previous line location is used as a clue for which region to focus on.

A masking function was also used to get an initial position for the left and right lines, by obscuring the left side of the image while searching for the right line, and vice versa.

An example of the line pixels selected is here:

![alt text][image6]

A 2nd order polynomial fit was then used to estimate the line. The numpy function polyfit() was used for this purpose. This logic is captured in my function get_lines(). The line pixels with the fit lines overlaid is shwon here:

![alt text][image7]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The fit lines were used to calculate the radius of curvature for the lines. This was done in the function measure_curvature(). This was evaluated at the point on the line closest to the bottom of the image. I also converted from pixels to meters based on known information of the lane width and the approximate length of the road visible in the camera (30 meters).

The position of the car with respect to the lane center was done in the function draw_position(). The assumption is that the camera is in the center of the car, so the center of the image can be treated as the center of the car. Then, it's just a matter of measuring how far that is from the left and right lines. This is also expressed in meters away from center.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The final image that has gone through all of the above, and with the lane plotted is shown here:

![alt text][image8]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result][video1]

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The most difficult part of issues I faced were creating the binary image, as I could choose from an endless number of combinations of thresholding techniques. I took inspiration from the course and from some classmates, and then spent time calibrating each function by tweaking thresholds until I was satisfied, and chose a simple combination of two techniques. The pipeline does have some trouble with dashed line. I could also see it failing whenever there is another line in the road that also meets my simple criteria, although a human can easily distinguish that from a real lane line. It's also very specific to the case of being in the middle of the lane - this would not effectively pick up additional lines, I don't think it's ready to handle osmething like changing lanes.

One idea I can see to make it more robust is to use additional techniques to produce the binary image, and for each frame, compare several techniques and choose the one that is the "best", based on recent data, number of pixels, etc.

More generally, with more time on the project I would really like to do a better job writing the actual code. I struggled with different approaches to parameterize the different algorithms, and have an inconsistent approach where sometimes I pass arguments, sometimes I rely on a global context, etc. If I was able to better encapsulate the functionality and make it easily testable, I could likely do a much better job of optimizing the pipeline as a whole. This seems like it would be a very valuable skill to learn, as it would apply to many situations where your model is made up of a combination of steps. 

Another idea for this is to use machine learning approaches to learn the optimal values for all of the parameters, as long as I had an effective way to evaluate the performance, which might be tough if there isn't a large amount of data out there contianing roads labeled with lane location.