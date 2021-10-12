---
layout: post
author: Lulin Zhang, Ewelina Rupnik, Marc Pierrot-Deseilligny
image: images/flowchart.png
toc: true
badges: true
comments: true
hide: false
search_exclude: true
categories: [Feature matching, Historical images, Multi-epoch, Pose estimation, Self-calibration]
description: "Feature matching for multi-epoch historical aerial images"
title: "Feature matching for multi-epoch historical aerial images" 
---


# Introduction

Historical images are a valuable source of information in analyzing the evolution of landscapes. To be able to fully exploit their potential, the images should be precisely calibrated with the help of feature correspondences.
Current state-of-the-art feature matching methods include SIFT (hand-crafted) and SuperGlue (deep learning). They work well on images within the same epoch (also refered to intra-epoch).

<p align="center">
  <img src="images/intra.png" width="800">
</p>

<p align="center">
Figure. Intra-epoch image pair
</p>

<p align="center">
  <img src="images/Homol-SIFT_OIS-Reech_IGNF_PVA_1-0__1970__C3544-0221_1970_CDP6452_1407.tifOIS-Reech_IGNF_PVA_1-0__1970__C3544-0221_1970_CDP6452_1408.tif.png" width="800">
</p>

<p align="center">
Figure. SIFT correspondences
</p>

<p align="center">
  <img src="images/Homol-SuperGlue_OIS-Reech_IGNF_PVA_1-0__1970__C3544-0221_1970_CDP6452_1407.tifOIS-Reech_IGNF_PVA_1-0__1970__C3544-0221_1970_CDP6452_1408.tif.png" width="800">
</p>

<p align="center">
Figure. SuperGlue correspondences
</p>

However, it is very challenging to recover correspondences in multi-epoch historical images due to:

* Drastic scene changes
* Low radiometric quality
* Deformation caused by scaning

<p align="center">
  <img src="images/Homol-SIFT-3DRANSAC-CrossCorrelation_OIS-Reech_IGNF_PVA_1-0__1970__C3544-0221_1970_CDP6452_1407.tifOIS-Reech_IGNF_PVA_1-0__1954-03-06__C3544-0211_1954_CDP866_0632.tif.png" width="800">
</p>

<p align="center">
Figure. Inter-epoch image pair
</p>

<p align="center">
  <img src="images/Homol-SIFT-3DRANSAC-CrossCorrelation_OIS-Reech_IGNF_PVA_1-0__1970__C3544-0221_1970_CDP6452_1407.tifOIS-Reech_IGNF_PVA_1-0__1954-03-06__C3544-0211_1954_CDP866_0632.tif.png" width="800">
</p>

<p align="center">
Figure. SIFT correspondences
</p>

SIFT recovered 0 correspondence. It is reasonable because SIFT is not sufficiently invariant over time.

<p align="center">
  <img src="images/Selection_533.png" width="800">
</p>

<p align="center">
Figure. SuperGlue correspondences
</p>

The result of SuperGlue seems not bad. However, the details revealed poor precision. This is not acceptable for high precision ground survey.

<p align="center">
  <img src="images/Selection_534.png" width="800">
</p>

<p align="center">
Figure. Our correspondences
</p>

Our method is capable to recover a large number of precise correspondences. The details will be introduced in next section.

# Methodology

<p align="center">
  <img src="images/flowchart.png" width="800">
</p>

<p align="center">
Figure. Full processing workflow
</p>

Our pipeline consists of 3 parts of processing: intra-epoch, inter-epoch and combined.

In the intra-epoch part, we process each epoch individually to get the image orientations and DSM.

The inter-epoch part is the key point of our pipeline. It is a rough-to-precise matching strategy.
Based on the image orientations and DSM from each epoch, we first match the DSMs to roughly co-register the 2 epochs.

<p align="center">
  <img src="images/tilematch.png" width="800">
</p>

<p align="center">
Figure. Workflow of the rough co-registration
</p>

We introduce tiling scheme and 

 then use it to guide a precise matching. We provide 

# Reference

-   ***L. Zhang***, E. Rupnik, M. Pierrot-Deseilligny, Feature matching for multi-epoch historical aerial images, ISPRS journal of photogrammetry and remote sensing. (https://linktobeadded)

#  Contact

> lulin.zhang.whu@gmail.com

> ewelina.rupnik@ign.fr

