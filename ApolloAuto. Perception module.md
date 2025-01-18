---
tags:
  - Robotics/Perception
  - Apollo
  - Sensor_Fusion
type: bookmark
source: https://github.com/ApolloAuto/apollo/blob/master/docs/06_Perception/perception_apollo_5.0.md
---

---
## Perception module

[Permalink: Perception module](https://github.com/ApolloAuto/apollo/blob/master/docs/06_Perception/perception_apollo_5.0.md#perception-module)   
The flow chart of Apollo 5.0 Perception module:   
![Apollo3.5\_perception\_detail.png](attachments/73a30f0b64c68572653c264111ac5750.png)    
To learn more about individual sub-modules, please visit [Perception - Apollo 3.0](https://github.com/ApolloAuto/apollo/blob/master/docs/06_Perception/perception_apollo_3.0.md)   

### Supports PaddlePaddle

[Permalink: Supports PaddlePaddle](https://github.com/ApolloAuto/apollo/blob/master/docs/06_Perception/perception_apollo_5.0.md#supports-paddlepaddle)   
The Apollo platform's perception module actively depended on Caffe for its modelling, but will now support PaddlePaddle, an open source platform developed by Baidu to support its various deep learning projects.

Some features include:   

- **PCNNSeg**: Object detection from 128-channel lidar or a fusion of three 16-channel lidars using PaddlePaddle   
- **PCameraDetector**: Object detection from a camera   
- **PlaneDetector**: Lane line detection from a camera   
   
### Using PaddlePaddle Features

[Permalink: Using PaddlePaddle Features](https://github.com/ApolloAuto/apollo/blob/master/docs/06_Perception/perception_apollo_5.0.md#using-paddlepaddle-features)   

1. To use the PaddlePaddle model for Camera Obstacle Detector, set `camera\_obstacle\_perception\_conf\_file` to `obstacle\_paddle.pt` in the following [configuration file](https://github.com/ApolloAuto/apollo/blob/master/modules/perception/production/conf/perception/camera/fusion_camera_detection_component.pb.txt)   
2. To use the PaddlePaddle model for LiDAR Obstacle Detector, set `use\_paddle` to `true` in the following [configuration file](https://github.com/ApolloAuto/apollo/blob/master/modules/perception/production/data/perception/lidar/models/cnnseg/velodyne128/cnnseg.conf)   
   
### Online sensor calibration service

[Permalink: Online sensor calibration service](https://github.com/ApolloAuto/apollo/blob/master/docs/06_Perception/perception_apollo_5.0.md#online-sensor-calibration-service)   
Apollo currently offers a robust calibration service to support your calibration requirements from LiDARs to IMU to Cameras. This service is currently being offered to select partners only. If you would like to learn more about the calibration service, please reach out to us via email: **[apollopartner@baidu.com](mailto:apollopartner@baidu.com)**   

### Manual Camera Calibration

[Permalink: Manual Camera Calibration](https://github.com/ApolloAuto/apollo/blob/master/docs/06_Perception/perception_apollo_5.0.md#manual-camera-calibration)   
In Apollo 5.0, Perception launched a manual camera calibration tool for camera extrinsic parameters. This tool is simple, reliable and user-friendly. It comes equipped with a visualizer and the calibration can be performed using your keyboard. It helps to estimate the camera's orientation (pitch, yaw, roll). It provides a vanishing point, horizon, and top down view as guidelines. Users would need to change the 3 angles to align a horizon and make the lane lines parallel.   
The process of manual calibration can be seen below:   
![Manual\_calib.png](attachments/2746c22f93e2a59abcb3a4d309e75e5d.png)    

### Closest In-Path Object (CIPO) Detection

[Permalink: Closest In-Path Object (CIPO) Detection](https://github.com/ApolloAuto/apollo/blob/master/docs/06_Perception/perception_apollo_5.0.md#closest-in-path-object-cipo-detection)   
The CIPO includes detection of key objects on the road for longitudinal control. It utilizes the object and ego-lane line detection output. It creates a virtual ego lane line using the vehicle's ego motion prediction. Any vehicle model including Sphere model, Bicycle model and 4-wheel tire model can be used for the ego motion prediction. Based on the vehicle model using the translation of velocity and angular velocity, the length and curvature of the pseudo lanes are determined.

Some examples of CIPO using Pseudo lane lines can be seen below:   

1. CIPO used for curved roads
   

    ![CIPO\_1.png](attachments/e0059edc95f7f3afb48fc934a4228e0a.png)    

2. CIPO for a street with no lane lines
   

    ![CIPO\_2.png](attachments/8f43ac78f5d49e9f942fb49a22741c5a.png)    

   
### Vanishing Point Detection

[Permalink: Vanishing Point Detection](https://github.com/ApolloAuto/apollo/blob/master/docs/06_Perception/perception_apollo_5.0.md#vanishing-point-detection)   
In Apollo 5.0, an additional branch of network is attached to the end of the lane encoder to detect the vanishing point. This branch is composed of convolutional layers and fully connected layers, where convolutional layers translate lane features for the vanishing point task and fully connected layers make a global summary of the whole image to output the vanishing point location. Instead of giving an output in `x`, `y` coordinate directly, the output of vanishing point is in the form of `dx`, `dy` which indicate its distances to the image center in `x`, `y` coordinates. The new branch of network is trained separately by using pre-trained lane features directly, where the model weights with respect to the lane line network is fixed. The Flow Diagram is included below, note that the red color denotes the flow of the vanishing point detection algorithm.   
![Vpt.png](attachments/ae570830d6d172a83c060e32af25e130.png)    
Two challenging visual examples of our vanishing point detection with lane network output are shown below:   

1. Illustrates the case that vanishing point can be detected when there is obstacle blocking the view:   
![Vpt1.png](attachments/f5b7a4b85fe1edec032e48f2f2e6e173.png)    
2. Illustrates the case of turning road with altitude changes:   
![Vpt2.png](attachments/070addf9b5e35166a7415c5cca66b469.png)    
   
### Key Features

[Permalink: Key Features](https://github.com/ApolloAuto/apollo/blob/master/docs/06_Perception/perception_apollo_5.0.md#key-features)   

- Regression to `(dx, dy)` rather than `(x, y)` reduces the search space   
- Additional convolution layer is needed for feature translation which casts CNN features for vanishing point purpose   
- Fully Connected layer is applied for holistic spatial summary of information, which is required for vanishing point estimation   
- The branch design supports diverse training strategies, e.g. fine tune pre-trained laneline model, only train the subnet with direct use of laneline features, co-train of multi-task network   
   
## Output of Perception

[Permalink: Output of Perception](https://github.com/ApolloAuto/apollo/blob/master/docs/06_Perception/perception_apollo_5.0.md#output-of-perception)   
The input of Planning and Control modules will be quite different with that of the previous Lidar-based system for Apollo 3.0.   

- Lane line output   
    - Polyline and/or a polynomial curve   
    - Lane type by position: L1(next left lane line), L0(left lane line), R0(right lane line), R1(next right lane line)   
- Object output   
    - 3D rectangular cuboid   
    - Relative velocity and direction   
    - Type: CIPV, PIHP, others   
    - Classification type: car, truck, bike, pedestrian   
    - Drops: trajectory of an object   
   

The world coordinate system is used as ego-coordinate in 3D where the rear center axle is an origin.   
If you want to try our perception modules and their associated visualizer, please refer to the [following document](https://github.com/ApolloAuto/apollo/blob/master/docs/06_Perception/how_to_run_perception_module_on_your_local_computer.md)   
