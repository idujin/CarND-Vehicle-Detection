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
[car_not_car_image]: ./examples/car_not_car.png
[HOG_image]: ./examples/HOG_example.png
[sliding_window_image]: ./examples/sliding_window.png
[output_bboxes_image]: ./examples/output_bboxes.png
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./result_project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Histogram of Oriented Gradients (HOG)

The code for this step is contained in the cell which is marked as 'Histogram of Oriented Gradients (HOG) feature extraction'.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is four examples of each of the `vehicle` and `non-vehicle` classes:

![alt text][car_not_car_image]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][HOG_image]

Beside of HOG feature, I used spatial features and histogram features. When I trained a linear SVM with or without spatial and histogram features using 500 samples, each of model achieved 99.5 percent and 98.0 percent of accuracy. The result shows that the performance of the model with spatial and histogram feature could be slightly higher than the model without the features.

I started with default parameter value provided in the class.
the model using the parameter chieved 99.07 percent of accuracy and I thought the performance of the model is quite acceptable. 
Here is the parameters I used: 

* Color space: 'YCrCb' 
* HOG orientations: 9   
* HOG pixels per cell: 8  
* HOG cells per block: 2 
* HOG channel: "ALL" 
* Spatial binning dimensions: (32, 32) 
* Number of histogram bins: 32   


### Sliding Window Search


To reject false positives, I first decided to restrict the search area on the image with ystart and ystop in the slide_window() function. On the restricted area, I slide the window with different scales 1.0, 1.5, 1.75 and 2.0. 
Here are some examples with different scale:

![alt text][sliding_window_image]


Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][output_bboxes_image]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video. 
The first method I tried is as below. 
From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap. I then assumed each blob corresponded to a vehicle. I constructed bounding boxes to cover the area of each blob detected. However, the above simple method could not be successfully applied to all frames because of false positives.
To reject false positives, I tried three additional strategies. First, I tried frame averaging on heatmap. At the same time, I collected a vehicle detection box in the same frames used in heatmap averaging for a later step. Second, after constructing bounding box using the averaged heatmap, I calculated the center and standard deviation of the detection box inside each bounding box. From the collected detection box, a new list of detection boxes was created by removing the box with a calculated standard deviation of the collected detection boxes greater than a certain value and a greater distance from the center of the vehicle detection box. Lastly, I calculated the heatmap from the newly created list of detection boxes and got the vehicle detection box.




Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

![alt text][image5]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image6]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image7]



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?


The major issue to implement my classifier is to detect false positives. Most false positives appeared in front-facing vehicles. However, since the front and rear of the vehicle were not separated and trained, I tried to eliminate false positives using appropriate thresholds and the tendency of each frame. After applying the additional method, the final video presented above was obtained.

