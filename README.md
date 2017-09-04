# **Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/example_images.png
[image2]: ./output_images/example_hogs.png
[image3]: ./output_images/out_img.png
[image4]: ./output_images/final_heat.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the code cells before [334] of the IPython notebook `P5.ipynb`.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image2]


#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and set the final color space as `YCrCb` with `spatial_size=(32, 32)`, `hist_bins=32`, HOG parameters `orientations=10`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)` for the best performance on the video clip.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using both color and HOG features with `hog_channel = 'ALL'` with the parameter setting above. I did horizatal flip for all training images to augment the dataset. I concatenated the different feature vectors then normalized them using `StandardScaler()` to prevent one feature from dominating others. The classification is done in code cell [414].

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

For video processing, I used dynamic window search (check function `vehicleDetection()` in the code cell [684] for details). Generally speaking, I search the two sides of the camera using large windows as they are where new cars could appear. If there are no cars detected at this moment, I also slide windows with smaller sizes over the possible area in front of the ego-car for 10 frames. After 10 frames if a car is detected, I will only search a small area around this car, which greatly decreases the computation and false positives. 

I used `cells_per_step=2` for 4 different scales of windows.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here is an example image:

![alt text][image3]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4). I also combined it with the previous lane detection project to obtain an enhanced experience. ;)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected. 

I run the heatmap method over 10 frames when there is no car detected on the image to minimize the false positives. Once a new car appears from the sides of the camera, shrink the search area to be only around the detected car(s).

Here's an example result showing the final detections and the heatmap from a frame of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the frame of video:

![alt text][image4]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

Firstly, the approach assumes the highway situations, where the positions of new cars are very predictable. So, I only detect the new cars from two sides then lock them. This would likely fail in some urban situations such as crossroads.

In addition, it can be seen that the detector mistakenly detects some cars from other sides of the road which won't cause any problem, but is unnecessary. If the detector can be combined with curb／road divider detection using camera or other sensors, once a car is on the left-most lane of highway, a lot of useless search can be saved. 

Finally, I spent a lot of time on getting rid of false positives. SVM with HOG is a simple and old-fashion way for object detection, hence not very robust. DPM and deep-learning-based approaches would be worth it to investigate if I were going to pursue the project further, and they could be quite helpful for minimization of false positives and robustness in more difficult road situations.

