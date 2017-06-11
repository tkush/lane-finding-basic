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

The two masks are then overlaid to get a composite mask for both lanes.

#### Edge Detection
With the above mask defined and applied to the original image, edge detection is next. This
consists of several steps including converting to grayscale, blurring and finally applying the Canny edge
algorithm. The thresholds for upper, lower and the kernel size for the Gaussian blur are provided as
global parameters and can be tuned.

![Binary mask (yellow + white) and edges detected](/images/img3.png)

#### Determining lane lines
First, a region of interest mask is applied on the edged image so only lines of interest are picked
up by the Hough transform. This is done by defining a polygon around the area of interest.

![Binary mask (yellow + white) and ROI mask applied](/images/img4.png)

To determine lane lines from the edge detected image, a probabilistic Hough transform is applied twice
on the image:
**Pass 1**: A high Hough tolerance is used to detect the strongest lines. The image produced is subtracted
from a copy of the input image so that the input to Pass 2 does not contain the strongest lines
**Pass 2**: A low Hough tolerance is used to detect the weaker lines. All the lines are then put together in a
single list.

![ROI masked image and strongest lane lines identified](/images/img5.png)

![ROI masked image(strong lines removed) and weaker lane lines identified](/images/img6.png)

#### Fit lines to find the lanes
The Hough transform returns a collection of lines based on start and end points. To get a single line for
each lane, I used numpy’s polyfit to fit all the start and end points through a single line. This line is then
cropped according to the ROI and drawn over the original image to depict the lane lines.

![Original image and lane lines identified](/images/img6.png)

#### Notes
This code can be improved in the following ways (this is not exhaustive :) ):
1. Automatic ROI detection: The Hough transform can be applied repetitively on a sample of the video to determine the best ROI automatically
2. Smoothing the lanes detected: Using information from the previous video frame, a weighted smoothing algorithm may be applied that smoothens out the lane lines detected in each frame
