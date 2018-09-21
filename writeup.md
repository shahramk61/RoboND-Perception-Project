

## Project: Perception Pick & Place
### Writeup Template: You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---


# Required Steps for a Passing Submission:
1. Extract features and train an SVM model on new objects (see `pick_list_*.yaml` in `/pr2_robot/config/` for the list of models you'll be trying to identify). 
2. Write a ROS node and subscribe to `/pr2/world/points` topic. This topic contains noisy point cloud data that you must work with.
3. Use filtering and RANSAC plane fitting to isolate the objects of interest from the rest of the scene.
4. Apply Euclidean clustering to create separate clusters for individual items.
5. Perform object recognition on these objects and assign them labels (markers in RViz).
6. Calculate the centroid (average in x, y and z) of the set of points belonging to that each object.
7. Create ROS messages containing the details of each object (name, pick_pose, etc.) and write these messages out to `.yaml` files, one for each of the 3 scenarios (`test1-3.world` in `/pr2_robot/worlds/`).  [See the example `output.yaml` for details on what the output should look like.](https://github.com/udacity/RoboND-Perception-Project/blob/master/pr2_robot/config/output.yaml)  
8. Submit a link to your GitHub repo for the project or the Python code for your perception pipeline and your output `.yaml` files (3 `.yaml` files, one for each test world).  You must have correctly identified 100% of objects from `pick_list_1.yaml` for `test1.world`, 80% of items from `pick_list_2.yaml` for `test2.world` and 75% of items from `pick_list_3.yaml` in `test3.world`.
9. Congratulations!  Your Done!

# Extra Challenges: Complete the Pick & Place
7. To create a collision map, publish a point cloud to the `/pr2/3d_map/points` topic and make sure you change the `point_cloud_topic` to `/pr2/3d_map/points` in `sensors.yaml` in the `/pr2_robot/config/` directory. This topic is read by Moveit!, which uses this point cloud input to generate a collision map, allowing the robot to plan its trajectory.  Keep in mind that later when you go to pick up an object, you must first remove it from this point cloud so it is removed from the collision map!
8. Rotate the robot to generate collision map of table sides. This can be accomplished by publishing joint angle value(in radians) to `/pr2/world_joint_controller/command`
9. Rotate the robot back to its original state.
10. Create a ROS Client for the “pick_place_routine” rosservice.  In the required steps above, you already created the messages you need to use this service. Checkout the [PickPlace.srv](https://github.com/udacity/RoboND-Perception-Project/tree/master/pr2_robot/srv) file to find out what arguments you must pass to this service.
11. If everything was done correctly, when you pass the appropriate messages to the `pick_place_routine` service, the selected arm will perform pick and place operation and display trajectory in the RViz window
12. Place all the objects from your pick list in their respective dropoff box and you have completed the challenge!
13. Looking for a bigger challenge?  Load up the `challenge.world` scenario and see if you can get your perception pipeline working there!

## [Rubric](https://review.udacity.com/#!/rubrics/1067/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation. 

[//]: # (Image References)

[image1]: ./writeup_pic/1.png
[image2]: ./writeup_pic/2.png
[image3]: ./writeup_pic/3.png
[image4]: ./writeup_pic/4.png
[image5]: ./writeup_pic/5.png
[image6]: ./writeup_pic/6.png
[image7]: ./writeup_pic/7.png
[image8]: ./writeup_pic/8.png
[image9]: ./writeup_pic/9.png
[image10]: ./writeup_pic/10.png

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Exercise 1, 2 and 3 pipeline implemented
#### 1. Complete Exercise 1 steps. Pipeline for filtering and RANSAC plane fitting implemented.
Steps taken for RANSAC filtering:

1- Converting ROS point cloud to PCL format.

2- Vox grid down-sampling : A voxel grid filter allows us to downsample the data by taking a spatial average of the points in the cloud confined by each voxel therefore reduce amount of data that needs to be processed without losing any information. For this project voxel leaf size of 0.01 seems to work the best.

Image below is noisy point cloud after Voxel downsampling:
![alt text](https://github.com/shahramk61/RoboND-Perception-Project/blob/master/writeup_pic/1.PNG)

3- PassThrough Filter : Passthrough filter is used to further reduce the amount of point cloud data that needs to be processed by cutting the unwanted data. For this project the z axis is limited between 0.6 and 1, also the y axis is limited between -0.4 and 0.4

4- statistical outlier filter: This filter is used to reduce the noise that exist within the point cloud data due to dust, humidity, etc. for this project number of neighboring points to analyze is set to 10(mean) and the std_dev to 0.1

5- RANSAC Plane Segmentation: This step was perform to identify the table and removing it from the point cloud. For this project SACMODEL-PLANE model was chosen using SAC-RANSAC method with max distance of 0.1 to identify the table.

Image below is object after removing the noise and the table:

![alt text][image2]

Image below is the table after removing the noise and the objects:

![alt text][image3]

#### 2. Complete Exercise 2 steps: Pipeline including clustering for segmentation implemented.  Steps taken for clustering:

1- removing the color information from point cloud data and extracting the spacial information

2- creating kdtree for Euclidean Cluster Extraction for this project cluster Tolerance is set to  0.05, Min cluster size is set 10 Max cluster size is set to 1000.

3- After each cluster is identified unique color is assigned to each cluster for visualization

Below is the image of clustered object with unique color assigned.

![alt text][image4]

#### 2. Complete Exercise 3 Steps.  Features extracted and SVM trained.  Object recognition implemented.
Steps to features extraction and SVM:

1- The object that will be encountered during the project are spawned and scanned in random and features such as HUE histogram and surface normal histogram are collected.

2- using the HUE histogram and surface normal histogram a linear SVM was trained. best result of achieved when 25 complete feature set was captured for all items and single SVM was trained to be used for all scene. 

3- The model then used with the features extracted from the object detected in the project to identify the objects.

Below are the confusion matrix:

![alt text][image5]

![alt text][image6]

![alt text][image7]

### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

1- world one: 100% of objects were identified correctly.

world 1 output yaml file:

```

object_list:
- arm_name: right
  object_name: biscuits
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.5422342419624329
      y: -0.24263377487659454
      z: 0.7103633284568787
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: -0.71
      z: 0.605
  test_scene_num: 1
- arm_name: right
  object_name: soap
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.5434260368347168
      y: -0.019366933032870293
      z: 0.6788225173950195
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: -0.71
      z: 0.605
  test_scene_num: 1
- arm_name: left
  object_name: soap2
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.445666640996933
      y: 0.22295624017715454
      z: 0.6812319755554199
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: 0.71
      z: 0.605
  test_scene_num: 1
  
```

Below is the image of world 1 table top:
![alt text][image8]


2- world two: four out of five objects were identified correctly.

world 2 output yaml file:
```
object_list:
- arm_name: right
  object_name: biscuits
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.5716516375541687
      y: -0.25037920475006104
      z: 0.7100716829299927
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: -0.71
      z: 0.605
  test_scene_num: 2
- arm_name: right
  object_name: soap
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.560656726360321
      y: 0.0035076900385320187
      z: 0.6792725324630737
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: -0.71
      z: 0.605
  test_scene_num: 2
- arm_name: left
  object_name: soap2
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.44518277049064636
      y: 0.22760513424873352
      z: 0.6800448298454285
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: 0.71
      z: 0.605
  test_scene_num: 2
- arm_name: left
  object_name: glue
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.631613552570343
      y: 0.13085141777992249
      z: 0.6830272078514099
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: 0.71
      z: 0.605
  test_scene_num: 2

```

Below is the image of world 2 table top:
![alt text][image9]



3- world three: 100% of the object were identified correctly.

world 2 output yaml file:
```
object_list:
- arm_name: left
  object_name: sticky_notes
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.43918490409851074
      y: 0.2170054018497467
      z: 0.6861549019813538
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: 0.71
      z: 0.605
  test_scene_num: 3
- arm_name: left
  object_name: book
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.4907659590244293
      y: 0.08218228071928024
      z: 0.7307093739509583
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: 0.71
      z: 0.605
  test_scene_num: 3
- arm_name: right
  object_name: snacks
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.42430976033210754
      y: -0.32636553049087524
      z: 0.7522720694541931
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: -0.71
      z: 0.605
  test_scene_num: 3
- arm_name: right
  object_name: biscuits
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.5902156233787537
      y: -0.21878312528133392
      z: 0.70981365442276
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: -0.71
      z: 0.605
  test_scene_num: 3
- arm_name: left
  object_name: eraser
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.6063997149467468
      y: 0.2827928960323334
      z: 0.6493993401527405
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: 0.71
      z: 0.605
  test_scene_num: 3
- arm_name: right
  object_name: soap2
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.4551470875740051
      y: -0.04460488259792328
      z: 0.6807572245597839
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: -0.71
      z: 0.605
  test_scene_num: 3
- arm_name: right
  object_name: soap
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.6786882877349854
      y: 0.005721858702600002
      z: 0.6788777112960815
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: -0.71
      z: 0.605
  test_scene_num: 3
- arm_name: left
  object_name: glue
  pick_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0.6123694181442261
      y: 0.14241153001785278
      z: 0.6850162744522095
  place_pose:
    orientation:
      w: 0.0
      x: 0.0
      y: 0.0
      z: 0.0
    position:
      x: 0
      y: 0.71
      z: 0.605
  test_scene_num: 3


```
world 3 output yaml file:

![alt text][image10]


### Challenges

Training different SVM model for each world seemed inefficient and time consuming therefore I just trained a single SVM model with all objects and used it for all three worlds. It worked better for me since training models with few classes and small amount of the noise in the detected object would sometimes cause miss identification. 
