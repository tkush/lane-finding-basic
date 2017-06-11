# Basic lane finding
## Keywords: Computer vision, Canny edges, OpenCV, Hough transform

#### Lane detection and overlay on video
This document describes the basic thoughts and workflow behind a simple lane detection algorithm.

#### Basic Ideas:
Create a pipeline that takes a video stream as an input and processes it frame by frame to do the
following:
1. Create white and yellow color masks to retain only the yellow and white regions in the frame. The assumption is that the lane lines are either yellow or white
2. Use Gaussian blur and Canny edge detection routines to determine edges in the image
3. Apply the masks in step 1 to the edge detected image
4. Apply a mask for a region of interest within the image. This is pre-determined and is supposed to be a region comprised of the lanes and leave out the rest.
5. Use a probabilistic Hough transform to determine lines in the image from step 4
6. Fit the best line through data received from the Hough transform and overlay this line on the video frame to depict the lane lines

In the following sections, I talk about how I approached some of the steps above.

#### Creating color masks
Since the lanes in all three videos are either yellow or white, I have chosen to create two separate color
masks for each color. For white, I simply apply color thresholding on the BGR image:

![Original and thresholded image (white)](/images/img1.png)

For yellow, thresholding using BGR was not proving to be robust between the input videos. So, I decided
to convert the image to the HSV color space and create a mask for yellow in this space instead. Note, in
the images below, only the left lane is yellow.

![Original (HSV) and thresholded image (yellow)](/images/img2.png)
