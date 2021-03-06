**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Also apply a color transform and append binned color features, as well as histograms of color, to the HOG feature vector. 
* Note: Normalize features
* Implement a sliding-window technique and use the trained classifier to search for vehicles in images.
* Run the pipeline on a video stream and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4


### Histogram of Oriented Gradients (HOG)

#### 1. Extracting HOG features from the training images.

Extracting HOG features is done in both `test.py` and `train.py` (for example in lines #137 through #149).  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Final choice of HOG parameters.

I tried various combinations of parameters of colorspace, number of orientations, and number of pixels per cell. All the combinations were used in a SVM classifier and showed high accuracy above 0.95 on the validation tests. In general, as I increased the number of orientations, the accuracy increased. However, the number of features significantly increased. This could cause overfitting, therefore I limited the number of orientations to 16 (and I fixed it at this number). Also, decreasing the number of pixels per cell significantly increased the number of HOG features, so I set the lower bound of at 8 (which was also showed very high accuracy. The color space selected is YCrCb, with all layers used. Using this color space, the number of false positives was low and easily filtered. Here is a summary of the paremeters used:

| Variable         		|     Description	        					| 
|:---------------------:|:-------------------------------:| 
| Color space		| YCrCb       			|  
| Orientations		| 16       			|  
| Pixels per cell		| 8      			| 
| Cell per block		| 2      			| 
| Hog channels		|  All       			|  


#### 3. Training a classifier using the selected HOG features.

I used a SVM classifier trained on the dataset provided with the project. The feature vectors are extracted as described above (HOG features only). The featuers are normalized. Twenty percent of the dataset is used for validation. The testing is done on the project video. The training is done in `train.py` in lines 175 and 178. 
This results in a 0.9905 validation accuracy for the classifier.


### Sliding Window Search

#### 1. Implementation.

I chose four window sizes: 48, 64, 92, 128. At each step of the search, the windows have 50% overlap. For each window size, the function `slide_window` used in `test.py` at line 179. A fixed box will slide over a specified region for each window size. For each box, HOG features are extracted and fed to the SVM classifier. If a car is detected, the box location will be cached.  


#### 2. Expample

Consider the following test image. 

![test1](original.png)

After applying a window search, the following hot windows are detected:

![test2](hot.png)

---

### Video Implementation

Here's a [link to my video result](https://www.youtube.com/watch?v=hL4NeaETh8U). A copy of this video (`full_video_labeled.mp4`).  

#### 2. Implemented a filter for false positives and some method for combining overlapping bounding boxes.

After finding the hot windows; i.e. the windows were cars are detected as in the image above; we notice that there are some false positives. These errors occur with minimum overlap between hot windows. So one way to get rid of them is to cache the intersection between multiple hot windows. In my code, at least three windows have to intersect. This is done via the functions `apply_threshold(heat,hot_windows)` and `np.clip(heat,2)`. To see the exact implementation look at lines 199 to 212 in `test.py`

### Here is a heatmap with filtered false positives:

![alt text](filtered_hot.png)

### Here is the final labels on the original image:
![alt text](filter_result.png)


---

### Discussion


The main problem was the large number of false positives, especially when the overlap between windows is large. Searching for the proper parameters to properly detect cars with low number of false positives was difficult. The pipeline is likely to fail in a new video with different lighting settings. This is simply due to the fact that I trained my classifier and chose the rest of parameters to work well on the project video. This is like training on the test set. To make the process more robust, we could use a deep learning classifier that accurately detects vehicles. I suspect that a deep learning approach will have fewer false positives. Also it does not require manual feature engineering. 

