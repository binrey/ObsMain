---
tags: 
type: bookmark
source: https://tsinghua-mars-lab.github.io/Occ3D/
---
---
##  Introduction

One of the most popular visual perception tasks is 3D object detection, their expressiveness is still limited as shown in Figure1:
(1) 3D bounding box representation erases the geometric details of objects, eg a bendy bus or a construction vehicle;
(2) rarely seen objects, like trash or tree branch on the streets, are often ignored and not label in existing datasets.
          
![Interpolate start reference image.](https://tsinghua-mars-lab.github.io/Occ3D/static/images/Occ3D_teaser.png) 
Figure 1:The Occ3D dataset demonstrates rich semantic and geometric expressiveness. 

We formalize the 3D occupancy prediction task as follows: a model needs to jointly estimate the occupancy state and semantic label of every voxel in the scene from images. 
The occupancy state of each voxel can be categorized as free, occupied, or unobserved. 
For occupied voxels, semantic labels are assigned. For objects that are not in the predefined categories, they are labeled as General Objects (GOs).



### 



##  CVPR2023 Occupancy Prediction Challenge

            To promote the 3D occupancy prediction task, we co-host a 3D Occupancy Prediction Challenge in CVPR2023.
            For more information about the challenge, please refer to  [the Challenge Page](https://github.com/CVPR2023-3D-Occupancy-Prediction/CVPR2023-3D-Occupancy-Prediction) .
          


### The Occ3D Dataset



### Dataset Downloads

            We generate two 3D occupancy prediction datasets,  [Occ3D-Waymo](https://drive.google.com/drive/folders/13WxRl9Zb_AshEwvD96Uwz8cHjRNrtfQk)             and  [Occ3D-nuScenes](https://drive.google.com/drive/folders/13WxRl9Zb_AshEwvD96Uwz8cHjRNrtfQk?usp=share_link) .
            Checkout our  [code and toolkit](https://github.com/Tsinghua-MARS-Lab/Occ3D.git)  to get started using our datasets. 
            Before using the dataset, you should agree to the terms of use of the
              [nuScenes](https://www.nuscenes.org/)  and 
              [Waymo](https://waymo.com/open/) .
             The code used to generate the data and the generated data are both subject to the  [MIT License](https://en.wikipedia.org/wiki/MIT_License) .
          
![Interpolate start reference image.](https://tsinghua-mars-lab.github.io/Occ3D/static/images/Dataset_new_statics.png) 
Table 1: Comparing Occ3D Datasets with other datasets for 3D perception.\



### Dataset Construction Pipeline

            Acquiring dense voxel-level annotations for a 3D scene
            can be challenging and impractical. To address this, we
            propose a semi-automatic label generation pipeline that utilizes existing labeled 3D perception datasets. 
          
![Interpolate start reference image.](https://tsinghua-mars-lab.github.io/Occ3D/static/images/pipeline_v3.pdf) 
Figure 2: Our label generation pipeline.
            Our proposed label generation pipeline addresses the above challenges, an overview is shown in Figure 2. Initially, in voxel densification, we increase the density of the point clouds by performing multi-frame aggregation for both static and dynamic objects separately. Then we employ a K-nearest neighbor algorithm to assign labels to unlabeled points and utilize mesh reconstruction to perform hole-filling. 
            Subsequently, we carry out occlusion reasoning from both LiDAR and camera perspectives, utilizing a ray-casting operation to label the occupancy state of each voxel. Finally, misaligned voxels are eliminated through an image-guided voxel refinement process.
          


### Quality Check

            Acquiring an occupancy representation that adheres to the complete shape of all objects is challenging.
            Therefore, evaluating the quality of the dataset and ensuring the effectiveness of each step in our pipeline is critical. 
          
![Interpolate start reference image.](https://tsinghua-mars-lab.github.io/Occ3D/static/images/quality_check.png) 
Figure 3: 3D-2D consistency.
            Our proposed label generation pipeline addresses the above challenges, an overview is shown in Figure 2. Initially, in voxel densification, we increase the density of the point clouds by performing multi-frame aggregation for both static and dynamic objects separately. Then we employ a K-nearest neighbor algorithm to assign labels to unlabeled points and utilize mesh reconstruction to perform hole-filling. 
            Subsequently, we carry out occlusion reasoning from both LiDAR and camera perspectives, utilizing a ray-casting operation to label the occupancy state of each voxel. Finally, misaligned voxels are eliminated through an image-guided voxel refinement process.
          


### Methods

            To deal with the challenging 3D occupancy prediction problem, we present a new transformer-based model named Coarse-to-Fine Occupancy (CTF-Occ).
            First, 2D image features are extracted from multi-view images with an image backbone. Then, 3D voxel
            queries aggregate 2D image features into 3D space via a
            cross-attention operation. Our approach involves using a
            pyramid voxel encoder that progressively improves voxel
            feature representations through incremental token selection
            and spatial cross-attention in a coarse-to-fine fashion. This
            method enhances the spatial resolution and fine-tunes the
            detailed geometry of objects, ultimately leading to more accurate 3D occupancy predictions.
          
![Interpolate start reference image.](https://tsinghua-mars-lab.github.io/Occ3D/static/images/occ_overall.png) 
Figure 4: The model architecture of CTF-Occ.
            An overview of CTF-Occ network is shown in Figure 3. First, 2D image features are extracted from multi-view images with an image backbone.
            Then, 3D voxel queries aggregate 2D image features into 3D space via a cross-attention operation.
          


### Experiments

            Our method outperforms previous methods on Occ3D-Nuscenes and Occ3D-Waymo dataset.
          
![Interpolate start reference image.](https://tsinghua-mars-lab.github.io/Occ3D/static/images/Occ3D-nuscenes_new.png) 
![Interpolate start reference image.](https://tsinghua-mars-lab.github.io/Occ3D/static/images/Occ3D-Waymo_new.png) 
Table 2: 3D occupancy prediction performance on the Occ3D-nuScenes and Occ3D-Waymo dataset. Cons. Veh represents
                  construction vehicle and Dri. Sur is for driveable surface.\
