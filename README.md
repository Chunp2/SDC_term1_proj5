

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector.
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/car_not_car.png
[image2]: ./output_images/HOG_example.png
[image3]: ./output_images/slidingwindows.png
[image4]: ./output_images/slidingwindow.png
[image5]: ./output_images/slidingwindow.png
[image6]: ./output_images/heatmap.png
[image7]: ./output_images/labels.png
[image8]: ./output_images/car_pos.png
[video1]: ./project_video_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---


#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the 4th code cell of the IPython notebook (`Project5.ipynb`).  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `Lab` color space and HOG parameters of `orientations = 9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and `orientations = 9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)` works the best.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using spatial, histogram, and HOG features. Before actually training those features, I defined a feature vector that stacked car and non car features in vertical, and I used StandardScaler function to fit

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I searched window positions at scales 1.0, 1.5, and 2.0 all over the image and came up with this result:

![alt text][image3]

I implemented a sliding window search in "DetectVehicle()" function under part "4.Vehicle Detection". For window scales, I initially used scales from 1.0 to 2.5 with 0.1 incrementation, but it took too long to compile the video. I experimented and came up with the scales (1.0, 1.5, 2.0) works the best. For window overlapping, I decided to skip 2 cells per each window slide, because skipping 1 and 3 cells per each window slide overlaps more or less than enough. Although skipping 1 cells per step covers across the image more thoroughly, there was no difference from 2 cells per step. And 3 cells per step was performing poorly compared to 2 cells per step. So I decided to pick 2 cells per step.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on five scales using Lab 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

![alt text][image5]
![alt text][image6]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image7]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image8]



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. First I used HOG, spatial binned, and histogram of color for extracting feature vectors to be trained by linear support vector machine. Initially I used YCrCb 3-channel HOG features, but I found that Lab 3-channel gives the better training accuracy compared to other color channels, such as YCrCb, HSV, YUV, and so on. In addition, I extracted features only from images in KITTI_extracted folder, but later this caused a problem that a white vehicle in test images and videos was not detected consistently. So I added more images from other sources such as GTI_Far, GTI_Left, GTI_MiddleClose, and GTI_Right, and this improved the result, detecting the white vehicle more consistently than solely training the extracted features from KITTI_extracted images. And it works fine detecting the vehicles in the test video, but it will fail in detecting vehicles under shadows. This can be improved by training images in diverse color channels to detect the shape and color of a vehicle more robust.
