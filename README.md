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

> This research was carried out within the [LaSTIG lab., IGN-ENSG, Univ. Gustave Eiffel.](https://www.umr-lastig.fr/)

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

We propose a fully automatic approach to computing robust inter-epoch feature correspondences. Our method consists of two steps: a rough co-registration by finding feature correspondences between DSMs (Digital Surface Model) derived within single epochs, and a precise feature matching on original RGB images. Our main contributions include:
* Rough-to-precise matching strategy that helps to drastically reduce ambiguity. In particular, we use the depth information to roughly co-register our epochs. The 3D landscape is globally stable over time and provides sufficient correspondences *through time*. Once co-registered, we levarage the 3D *a priori* to narrow down the search space in precise matching. 
* Upscaling of the learning based feature matching algorithms to high resolution imagery. To do that, we introduced an image tiling scheme.

In the following we present the methodology and some experiments. If you are interested in using our method, please refer to the source code of MicMac Github [^4], as well as 2 jupyter tutorials [^5] [^6]. We also provide an introduction video [^7]. The datasets used in our publication [^3] will be soon published in an open-access repository.

# Methodology

<p align="center">
  <img src="images/flowchart.png" width="800">
</p>

<p align="center">
Figure 2. Full processing workflow
</p>

Our pipeline consists of 3 parts of processing: ***intra-epoch***, ***inter-epoch*** and combined. For the sake of simplicity, we only exhibit the processing flow of two epochs, however, it can be easily extended to more epochs. The inter-epoch part contains the key developments of our pipeline. It matches the DSMs resulted from intra-epoch processing to roughly co-register 2 epochs, and uses it to narrow down the searching space for precise matching. The resulting correspondences are then used to refine the orientations in the combined processing (i.e., bundle adjustement).


## Rough co-registration

Based on the image orientations and DSM from each epoch, we match the DSMs to roughly co-register the 2 epochs. The workflow is displayed below.

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

We provide 2 contributions to get robust result:

(1) We introduce tiling scheme and rotation hypotheses to improve the matching performance, as SuperGlue provides unsatisfactory result on large images and it is not invariant to rotations larger than 45◦.

(2) We choose matching DSMs instead of original RGB images for the following merits:
* Redundancy caused by the forward and side overlapping areas is removed;
* It implicitly enables a follow-up search for globally consistent inliers;
* It decreases the combinatorial complexity caused by rotation ambiguity;
* Even under important scene changes, DSMs generally provide stable information over time.

We also considered using orthophotos, however, fewer matches are found because the scene radiometry change is more pronounced than the 3D landscape change.

We display an example to show how our tiling scheme improved the performance of SuperGlue, as well as the comparison between orthophotos and DSMs:

|(a) Multi-epoch orthophoto pair|(b) Multi-epoch DSM pair|
|-|-|
|<p align="center"> <img src="images/DOMoverlapping.png" width="600"></p>|<p align="center"> <img src="images/DSMoverlapping.png" width="600"></p>|
|<p align="center">(c) SuperGlue correspondences on orthophotos</p>|<p align="center">(d) SuperGlue correspondences on DSMs</p>|
|<p align="center"> <img src="images/DOM-SuperGlue.jpg" width="600"> </p>|<p align="center"> <img src="images/DSM-SuperGlue.png" width="600"> </p>|
|<p align="center">(e) Our correspondences on orthophotos</p>|<p align="center">(f) Our correspondences on DSMs</p>|
|<p align="center"> <img src="images/DOM-Ours.png" width="600"> </p>|<p align="center"> <img src="images/DSM-Ours.png" width="600"> </p>|
<p align="center">Figure 5. Comparison of co-registration with orthophotos and DSMs. (a-b) Red rectangles indicate the overlapping area.

As can be seen, (1) for both orthophotos and DSMs, the correspondences of SuperGlue are all wrong, while ours (SuperGlue combined with tiling scheme, followed by RANSAC) recovered a large number of good correspondences. (2) DSMs have more inliers than orthophotos, because there is much less perceived change in the DSMs than there is in the orthophotos.

 > Note: For more details, please refer to our publication [^3].

## Precise matching

We perform a precise matching under the guidance of the co-registered DSMs.

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

# Lesson learned

## 1. It is useful to adopt *a priori* to reduce ambiguity.

As multi-epoch images often display very different appearance, it helps to exploit *a priori* (e.g. depth information extracted from single epoch) to narrow down the searching space. The *a priori* could be used in as many ways as possible. There are at least 3 ways to take advantage of the depth information:
* matching depth images for rough co-reigstration.
* use co-registered depth information to handle scale and rotation difference in 2 ways: (1) for deep learning matching methods such as SuperGlue, get patch pairs free of scale and rotation difference; (2) for hand-crafted mehtods such as SIFT, reject candidate matches whose scales and rotations computed by
SIFT are incoherent with prediction by depth.
* filter candidate matches by projecting them onto depth to get 3D points, followed by running RANSAC on 3D Helmert transformation model.

## 2. Deep learning feature matching methods underperform on high resolution images, it can be improved with tiling scheme.

Deep learning feature matching methods often provide inaccurate features for different reasons:
* network trained on small images in order to perform in real-time, with SuperGlue as a representative;
* features extracted on CNN feature maps with limited spatial resolution, with D2-Net as a representative.

We can obtain accurate features on high resolution images by cropping the input images into tiles, and match the tile pairs in 2 possible ways:
* If no *a priori* is available, it is neccessary to match all the potential tile pairs;
* If *a priori* is available (e.g. the depths are roughly co-registered), it can be used to predict the tile from one image to another.

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
