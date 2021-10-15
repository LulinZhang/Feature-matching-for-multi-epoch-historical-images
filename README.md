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

Historical imagery is characterized by high spatial resolution and stereoscopic acquisitions, providing a valuable resource for recovering 3D land-cover information. Accurate geo-referencing of diachronic historical images by means of self-calibration remains a bottleneck because of the difficulty to find sufficient amount of feature correspondences under evolving landscapes. As shown in Figure 1 (c-d), current *state-of-the-art* feature matching methods (including SIFT [^1] and SuperGlue [^2]) perfom well on images acquired within the same epoch (also refered to as intra-epoch). Yet, due to drastic scene changes and heterogeneous acquisition conditions, they underperform to find feature correspondences across different epochs (also referred to as inter-epoch) Figure 1 (e-f), while our method [^3] is able to recover numerous and accurate correspondences Figure 1 (g).

|         (a) Intra-epoch image pair        | (b) Inter-epoch image pair |
|-|-|
| <p align="center"> <img src="images/intra.png" width="400"> </p> | <p align="center"> <img src="images/Homol-SIFT-3DRANSAC-CrossCorrelation_OIS-Reech_IGNF_PVA_1-0__1970__C3544-0221_1970_CDP6452_1407.tifOIS-Reech_IGNF_PVA_1-0__1954-03-06__C3544-0211_1954_CDP866_0632.tif.png" width="400"> </p> |
|   <p align="center">   (c) ***SIFT*** : good correspondences </p> |      <p align="center">    (d) ***SIFT*** : 0 correspondences  </p> |
| <p align="center"> <img src="images/Homol-SIFT_OIS-Reech_IGNF_PVA_1-0__1970__C3544-0221_1970_CDP6452_1407.tifOIS-Reech_IGNF_PVA_1-0__1970__C3544-0221_1970_CDP6452_1408.tif.png" width="400"> </p> | <p align="center">  <img src="images/Homol-SIFT-3DRANSAC-CrossCorrelation_OIS-Reech_IGNF_PVA_1-0__1970__C3544-0221_1970_CDP6452_1407.tifOIS-Reech_IGNF_PVA_1-0__1954-03-06__C3544-0211_1954_CDP866_0632.tif.png" width="400"> </p> |
|  <p align="center">   (e) ***SuperGlue*** : good correspondences </p>     |   <p align="center">   (f) ***SuperGlue*** : inaccurate correspondences  </p>       |
| <p align="center"> <img src="images/Homol-SuperGlue_OIS-Reech_IGNF_PVA_1-0__1970__C3544-0221_1970_CDP6452_1407.tifOIS-Reech_IGNF_PVA_1-0__1970__C3544-0221_1970_CDP6452_1408.tif.png" width="400"> </p> | <p align="center">  <img src="images/Selection_533.png" width="300"> </p> |
| | <p align="center"> (g) ***Ours*** : accurate correspondences </p> |
| |<p align="center"> <img src="images/Selection_534.png" width="300"> </p>|

<p align="center"> Figure 1. SIFT, SuperGlue and Our features computed on an intra-epoch (left) and inter-epoch (right) image pair. The blue line connecting 2 points from the left and right images represent the feature correspondences. </p> 

In this work we propose a fully automatic approach to computing dense and robust inter-epoch feature correspondences. Our method consists of a rough co-registration by finding feature correspondences between DSMs derived within single epochs, and a precise feature matching on original RGB images. Our main contributions include:
* Exploiting rough-to-precise matching strategy to reduce ambiguity: (1) use depth information to get robust rough co-registration as the 3D landscape often stays globally stable over time; (2) use co-registered depth information for prediction to narrow down search space.
* Upscale the matching performance for both hand-crafted and deep learning methods: (1) for hand-crafted method, check scale and rotation of keypoints to reduce ambiguity; (2) for deep learning method, introduce tiling scheme to improve precision for high resolution images.

In the follwoing the methodology as well as experiments are demenstrated. 

If you are interested in using our method, please refer to the source code of MicMac Github [^4], as well as 2 jupyter tutorials [^5] [^6] to familiarize you with the usage. We also provide an introduction video [^7].

The datasets used in our publication [^3] will be soon published in an open-access repository.

# Methodology

<p align="center">
  <img src="images/flowchart.png" width="800">
</p>

<p align="center">
Figure 2. Full processing workflow
</p>

Our pipeline consists of 3 parts of processing: intra-epoch, inter-epoch and combined.

In the intra-epoch part, we process each epoch individually to get the image orientations and DSM.

The inter-epoch part is the key point of our pipeline. It is a rough-to-precise matching strategy.

In the combined processing, we use the correspondences to refine the orientations.

The details of the inter-epoch processing are explained below:

## Rough co-registration

Based on the image orientations and DSM from each epoch, we first match the DSMs to roughly co-register the 2 epochs.

As SuperGlue provides unsatisfactory result on large images and it is not invariant to rotations larger than 45◦, we introduce tiling scheme and rotation hypotheses to improve the performance of SuperGlue.

<p align="center">
  <img src="images/tilematch.png" width="800">
</p>

<p align="center">
Figure 3. Workflow of the rough co-registration
</p>

<p align="center">
  <img src="images/rotation.png" width="800">
</p>

<p align="center">
Figure 4. Four rotation hypotheses
</p>

> Note: For more details, please refer to our publication [^3].

We display an example to show how our tiling scheme improved the performance of SuperGlue:

<p align="center">
  <img src="images/DSMoverlapping.png" width="600">
</p>

<p align="center">
Figure 5. Multi-epoch DSM pair (Left and right are DSMs of epoch 1954 and 2014 individually. Red rectangles indicate the overlapping area.)
</p>

<p align="center">
  <img src="images/DSM-SuperGlue.png" width="600">
</p>

<p align="center">
Figure 6. Correspondences of SuperGlue
</p>

<p align="center">
  <img src="images/DSM-Ours.png" width="600">
</p>

<p align="center">
Figure 7. Correspondences of ours
</p>

As can be seen, the correspondences of SuperGlue are all wrong, while ours (SuperGlue combined with tiling scheme, followed by RANSAC) recovered a large number of good correspondences.

## Precise matching

We perform a precise matching under the guidance of the co-registered DSM.

We provide 2 options (guided matching and patch matching) to get tentative inter-epoch correspondences, then remove the outliers with 3D RANSAC and cross correlation.

### Get tentative inter-epoch correspondences
Guided matching is designed for hand-crafted methods like SIFT. We predict the keypoint locaiton from one epoch to another via the co-registered DSM, and search only the neighborhood of the predicted keypoint to reduce ambiguity.

Patch matching is designed for deep learning methods like SuperGlue. We use the co-registered DSM to predict the corresponding patches, followed with resampling to remove the scale and rotation difference. The patch pair will be feed into SuperGlue to get tentative correspondences.

<p align="center">
  <img src="images/precisematch.png" width="800">
</p>

<p align="center">
Figure 8. Guided matching and patch matching
</p>

### Get enhanced inter-epoch correspondences

We project the tentative correspondences onto respective DSMs to get 3D points, and remove outliers in a RANSAC routine based on 3D
spatial similarity model.

<p align="center">
  <img src="images/3DRANSAC.png" width="300">
</p>

<p align="center">
Figure 9. 3D RANSAC
</p>

### Get final inter-epoch correspondences

We apply cross-correlation for final validation. Feature correspondences with their correlation scores below a predefined threshold
are discarded.
The rectangles in the picture below represent the cross-correlation window, false match (red) is eliminated, while true match (blue) is kept.

<p align="center">
  <img src="images/tiept.png" width="400">
</p>

<p align="center">
Figure 10. Cross-correlation
</p>

# Experiment

We test our method on a dataset with drastic scene changes displayed below:

|         (a) Epoch 1954        |         (b) Epoch 2014        |
|-|-|
| <p align="center">  <img src="images/1954.png" width="450"> </p> |<p align="center"> <img src="images/2014.png" width="450"> </p> |

<p align="center">
Figure 11. Dataset with drastic scene changes
</p>

We recover inter-epoch correspondences and refine the image orientations, then calculate DSMs in each epoch and adopt the conception of DoD (Difference of DSMs) for evaluation.

Ideally, the DoD should only display the scene changes as shown in the picture below.

<p align="center">
  <img src="images/IdealDoD.png" width="600">
</p>

<p align="center">
Figure 12. DoD in ideal case
</p>

But in real case, a doom effect as shown below would appear due to poorly estimated camera parameters.

<p align="center">
  <img src="images/RealDoD.png" width="600">
</p>

<p align="center">
Figure 13. DoD in real case
</p>

We display 4 sets of DoDs below.

(1) DoD resulted from orientations based on 3D helmert transformation using 3 manually measured GCPs. 

This DoD is for comparison. As can be seen, this DoD showed obvious doom effect as the camera parameters of epoch 1954 are poorly estimated;

(2) DoD resulted from orientations based on 3D helmert transformation using GCPs automatically recovered from our rough co-registration. 

The systematic error is mitigated thanks to our dense correspondences in DSMs;

(3) DoD resulted from orientations refined in bundle adjustment with our correspondences based on guided matching;

(4) DoD resulted from orientations refined in bundle adjustment with our correspondences based on patch matching.

In the DoD of (3) and (4), the doom effect is effectively mitigated while the real scene changes are kept, thanks to our numerous and precise inter-epoch correspondences.

<p align="center">
  <img src="images/DoD.png" width="800">
</p>

<p align="center">
Figure 14. DoD in real case
</p>

A subregion of scene evolution is displayed below, a seaport as well as several new buildings emerged and are well detected with our method.

<p align="center">
  <img src="images/sceneEvolution.png" width="600">
</p>

<p align="center">
Figure 15. Scene evolution
</p>

# Conclusion

Our method exploited rough-to-precise matching strategy to reduce ambiguity with the help of the depth information. We also introduced tiling scheme and checking scale and rotation to upscale the matching performance.

Experiments showed that our method is able to mitigate systematic errors induced by poorly estimated camera, and it is robust to drastic scene changes.

#  Contact

> lulin.zhang.whu@gmail.com

> ewelina.rupnik@ign.fr

# Reference

[^1]: D. G. Lowe, Distinctive image features from scale-invariant keypoints, International journal of computer vision 60 (2) (2004) 91–110.

[^2]: P.-E. Sarlin, D. DeTone, T. Malisiewicz, A. Rabinovich, Superglue: Learning feature matching with graph neural networks, in: Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, 4938–4947, 2020.

[^3]:  ***L. Zhang***, E. Rupnik, M. Pierrot-Deseilligny, [Feature matching for multi-epoch historical aerial images](https://linktobeadded), ISPRS journal of photogrammetry and remote sensing

[^4]: MicMac, GitHub, https://github.com/micmacIGN/micmac/tree/master/src/uti_phgrm/TiePHistorical, 2021.

[^5]: [Jupyter tutorial of our method for matching multi-epoch historical aerial images](https://colab.research.google.com/drive/1poEXIeKbPcJT_2hyQOBhzcj1EEhO8OgD)

[^6]: [Jupyter tutorial of our method for matching multi-epoch historical images (aerial and satellite images mixed)](https://colab.research.google.com/drive/14okQ8bBhEZmy6EGRIQvazTqrN39oc_K5)

[^7]: [Introduction video for "Feature matching for multi-epoch historical aerial images"](https://linktobeadded)
