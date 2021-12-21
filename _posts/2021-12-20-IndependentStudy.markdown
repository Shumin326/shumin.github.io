---
layout: post
title:  "Robot Indoor Navigation with Architectural Maps"
date:   2021-12-20 20:21:54 -0600
categories: jekyll update
usemathjax: true
---

## Overview
Mapping and localization algorithms require a manual walkthrough and capture of the environment. Upon loop closure they are ready to navigate with algorithms like SLAM. This project’s goal is to improve this process for indoor spaces with two steps: 

1. Cheap initial navigable map, e.g. floor plans. Use architectural maps to slow the cost and increase the speed of the initial map capture by eliminating the manual walkthrough. We aim to accomplish this by transforming the building’s blueprint into a 3D navigable world with occupancy map and semantic landmarks within which we do path planning and localization. 
2. Landmark detection: We implemented a landmark detection model so that robot can recognize landmarks in real world navigation. Those landmarks are further utilized to improve localization performance.

This independent study developed a complete pipeline for robots navigation under indoor environments, e.g. hospitals and offices, with only an architectural map. 

The overall workflow is shown in the figure below. Given a building blueprint, e.g. floor plans, the Auto Mapping module converts the floor plan into an occupancy map and a list of landmarks with accurate location and orientation. The planning module determines a navigation strategy for robots to navigate and has the potential to apply onto multi-agent navigation applications. It first lets the user manually determine a set of sparse waypoints in order to make sure that LiDAR scans cover every corner of the building. Then it uses a Pure Pursuit controller and FMT* path planner to navigate among those waypoints. Graph partitioning algorithm and closed TSP solution are used to make sure all waypoints are visited only once. The landmark detection module detects landmarks and also shares this information in the format of labeled bounding boxes with the localization module. The Localization module uses particle filter to fuse sensor data, i.e. lidar scan, odometry and landmarks, and obtain an estimated pose.

<img src="https://i.ibb.co/5cpPZ1V/system-overview.png" alt="system-overview" border="0">

## Localization
The localization part is the main focus of this project. It aims at providing a robust and accurate pose estimate for other modules(planning and control, etc) to function properly. The localization module takes as input the laser scans, noisy odometry and landmarks, and fuses them with a particle filter(alg. 1). There are mainly three updates happening inside the loop:
1. Motion model update using odometry. This step updates the particle poses by dead reckoning the odometry data and will have accumulated error overtime. Since the odometry used in onboard is quite noisy, the motion model update was manually added some gaussian noise.
2. Sensor update with laser scan. This step updates the particle weights by comparing the obtained ranges from laser scan with the ray marching ranges computed by RangeLibc and the reference map. 
3. Sensor update with landmarks. This step updates the particle weights by doing a landmark projection and matching(shown in Alg.2) algorithm which compares the observed landmarks with reference landmarks. More details will be provided in the next section.

<img src="https://i.ibb.co/68YNY9z/algo1.png" alt="algo1" border="0">


## Landmark Detection
The landmark detection is developed using a well known, high performance and highly customizable object detection model: YOLO. Since our indoor environment is vastly different from its training dataset environments where images are collected mostly outdoors and from a human perspective, a customized dataset was created for fine-tuning the YOLO model. The trained model can perform very robust and fast indoor landmark detection which is exactly what we needed for this project.

<img src="https://i.ibb.co/KWFN2sT/landmark-detection.png" alt="landmark-detection" border="0">

## Landmark Projection and Matching
The detected landmarks as bounding boxes can be used to further enhance the localization performance because we already have certain landmarks positions and orientations embedded in the floor maps, e.g. doors and windows. Thus, this section introduces the landmark projection and matching algorithm(alg. 2) that uses reference landmarks of floor maps and detect landmark in real time to update particle weights and improve the robustness of localization module especially at low-feature places such as the narrow hallway.

<img src="https://i.ibb.co/GJHNcCd/alg2.png" alt="alg2" border="0">

Determining the “seeable” landmarks is to check whether a landmark satisfies the three conditions(also shown in the figure below).  
1. It’s not blocked by any obstacle 
2. It’s within camera’s sight distance
3. It’s within the camera’s field of view(FOV). 

The first requirement is ensured by comparing the ray marching result from robot to the landmark and the actual distance between them two. If they’re equal(with some tolerance) then the landmark is not blocked by obstacles. The second and third requirements are ensured by computing the distance and angles between robot and landmark and compare with sight distance as well as camera FOV.

<img src="https://i.ibb.co/3SqYh4n/seeable.png" alt="seeable" border="0">

Seeable landmarks are in world frame but the detected landmarks are in camera frame, so we need to transform the seeable landmarks from world to camera frame. For simplicity, the frames are denoted as world(w), Robot(R) and Camera(C):

<img src="https://i.ibb.co/PgNMT8s/Eq1.png" alt="Eq1" border="0">

Where Lw  is the landmark in world frame and LC  is the converted landmarks in the camera frame. The transformation matrices can be obtained using particle pose pi=(x,y,theta) and camera extrinsic matrix E and intrinsic matrix I.

<img src="https://i.ibb.co/0Vm0k2t/eq2.png" alt="eq2" border="0">

Up to now we’ve found the set of expected landmarks to be seen in the camera frame and we’re ready to compare them with detected landmarks L. Since the number of two sets are not necessarily equal, use K-Nearest-Neighbor to find matches and compute the average distance d. Then update the particle weight w:

<img src="https://i.ibb.co/6mrxQt0/eq3.png" alt="eq3" border="0">

Note that in order to make the computation less stressful so it can be run in real time, the particle filter algorithm is vectorized instead of doing a loop for each particle, the latter is just to make the algorithm easier to understand.

## Code and Demo Videos
The [mLab github](https://github.com/mlab-upenn) contains the repos for all the modules including localization, landmark detection, as well as planning module and Auto-mapping module. They're for the whole project "CAD2CAV: From Building Blueprints to Scalable Multi-robot navigation".

in specific, the repos for my independent study project are:
1. [Localization module](https://github.com/shineyruan/particle_filter).
2. [Landmark detection module](https://github.com/Shumin326/darknet_ros). It also contains instructions on fine tuning as well as a link to my model.


The demo videos can be viewed in the repos(listed above) README as well as this [youtube video list](https://www.youtube.com/playlist?list=PL8Fip0E9YRSuZok9umO4fMpGWx_UsfoaL).
