---
title: "DeepURL: Deep Pose Estimation Framework for Underwater Relative Localization"
collection: publications
permalink: /publications/deepurl
date: 2020-07-30
venue: 'IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)'
paperurl: 'https://arxiv.org/abs/2003.05523'
citation: '<b> Bharat Joshi </b>, Md Modasshir, Travis Manderson, Hunter Damron, Marios Xanthidis, Alberto Quattrini Li,
 Ioannis Rekleitis, Gregory Dudek. accepted to <i> Proceedings IEEE/RSJ International Conference on Intelligent Robots and Systems </i>. 
 <b> IROS 2020 </b>'
bibtex: '/bibtex/deepurl.txt'
---

## Abstract

In this paper, we propose a real-time deep learning approach for determining the 6D relative pose of Autonomous 
Underwater Vehicles (AUV) from a single image. A team of autonomous robots localizing themselves in a communication-constrained 
underwater environment is essential for many applications such as underwater exploration, mapping, multi-robot convoying, 
and other multi-robot tasks. Due to the profound difficulty of collecting ground truth images with accurate 6D poses underwater, 
this work utilizes rendered images from the Unreal Game Engine simulation for training. An image-to-image translation network is 
employed to bridge the gap between the rendered and the real images producing synthetic images for training. 
The proposed method predicts the 6D pose of an AUV from a single image as 2D image keypoints representing 8 corners of the 
3D model of the AUV, and then the 6D pose in the camera coordinates is determined using RANSAC-based PnP. 
Experimental results in real-world underwater environments (swimming pool and ocean) with different cameras demonstrate the 
robustness and accuracy of the proposed technique in terms of translation error and orientation error over the state-of-the-art methods. 
The code  is publicly available.

<iframe width="300" height="200" src="https://www.youtube.com/embed/gh6iDQmETaM"> </iframe>
