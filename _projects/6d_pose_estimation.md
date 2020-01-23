---
title: "Object Detection driven 6D Pose Estimation using Synthetic Simulated images in
Underwater Domain"
collection: publications
permalink: /projects/deepcl
date: 2019-11-24
venue: 
paperurl: 
citation: 
bibtex: 
---

## Abstract
In recent works estimating 6D pose by training neural networks to predict the 2D locations of 3D keypoints, 
from which the pose can be obtained using a PnP algorithm have gained popularity. Underwater domain presents a significant
challenge in collecting training data with the corresponding 6-DoF poses accurately and underwater images suffer from
distortion, color loss, and poor quality. To overcome this expense of data collection in the challenging environment,
the proposed method is trained on simulated data generated with Unreal Game Engine where the CAD model of Aqua AUV is
projected over the sample sea images.

The proposed method uses object detection based 6D pose estimation framework where visible parts of the object
contribute to local pose prediction in the form of 2D keypoint locations. Then, we use the predicted confidence
to combine these pose coordinates into 3D-to-2D correspondences and use RANSAC to find the relative pose from the camera.
The network architecture is based on the YOLO framework and can achieve real-time performance.

<iframe src="https://www.youtube.com/embed/uPOJwQ-1bRs"> </iframe>


    