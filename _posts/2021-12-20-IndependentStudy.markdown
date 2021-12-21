---
layout: post
title:  "Robot Indoor Navigation with Architectural Maps"
date:   2021-12-20 20:21:54 -0600
categories: jekyll update
---

## Overview
Mapping and localization algorithms require a manual walkthrough and capture of the environment. Upon loop closure they are ready to navigate with algorithms like SLAM. This project’s goal is to improve this process for indoor spaces with two steps: 

1. Cheap initial navigable map, e.g. floor plans. Use architectural maps to slow the cost and increase the speed of the initial map capture by eliminating the manual walkthrough. We aim to accomplish this by transforming the building’s blueprint into a 3D navigable world with occupancy map and semantic landmarks within which we do path planning and localization. 
2. Landmark detection: We implemented a landmark detection model so that robot can recognize landmarks in real world navigation. Those landmarks are further utilized to improve localization performance.

This independent study developed a complete pipeline for robots navigation under indoor environments, e.g. hospitals and offices, with only an architectural map. 

The overall workflow is shown in the figure below. Given a building blueprint, e.g. floor plans, the Auto Mapping module converts the floor plan into an occupancy map and a list of landmarks with accurate location and orientation. The planning module determines a navigation strategy for robots to navigate and has the potential to apply onto multi-agent navigation applications. It first lets the user manually determine a set of sparse waypoints in order to make sure that LiDAR scans cover every corner of the building. Then it uses a Pure Pursuit controller and FMT* path planner to navigate among those waypoints. Graph partitioning algorithm and closed TSP solution are used to make sure all waypoints are visited only once. The landmark detection module detects landmarks and also shares this information in the format of labeled bounding boxes with the localization module. The Localization module uses particle filter to fuse sensor data, i.e. lidar scan, odometry and landmarks, and obtain an estimated pose.

![system Overview](/img/IndependentStudy/system_overview.png)
