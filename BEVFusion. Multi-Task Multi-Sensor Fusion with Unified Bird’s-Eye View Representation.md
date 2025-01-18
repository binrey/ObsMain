---
tags:
  - Sensor_Fusion
  - Object_Detection/3D
  - Segmentation/BEV
type: paper
published: 2022
source: 
inferModality:
  - L
  - C
  - LC
fps: 8.5
---

---

![[attachments/2205-13542v2.pdf]]

## Problems and Solutions
### Inefficiency of Point-Level Sensor Fusion
**Problem**: Traditional multi-sensor fusion methods often rely on **point-level fusion**, which augments the LiDAR point cloud with features from cameras. This approach is inefficient and can be **computationally expensive**.   
**Solution**: The paper introduces a **unified Bird’s-Eye View (BEV) representation**, where features from different sensors (e.g., cameras, LiDAR) are fused in a shared BEV space. This **task-agnostic** approach preserves both **geometric** (from LiDAR) and **semantic** (from cameras) information without introducing the inefficiencies of point-level fusion. The authors also optimize the BEV pooling step, reducing the runtime of this transformation by more than 40×.   
### Loss of Semantic and Geometric Information in Sensor Fusion
**Problem**: Previous methods that project camera features onto LiDAR points suffer from **loss of semantic information** (only 5% of the camera features can be mapped to LiDAR points). This is especially problematic for **semantic-oriented tasks** like BEV map segmentation.   

![[attachments/demo-1 1.gif]]

![[attachments/image_r 1.png]]

**Solution**: BEVFusion maintains **dense semantic information** from the camera images and combines it with the **geometric accuracy** of LiDAR. This is achieved by projecting camera features into the BEV space, keeping the full semantic density of the image data, while avoiding the geometric distortion caused by traditional camera-to-LiDAR projections.   

### **High Computational Cost in Multi-Sensor Fusion**
**Problem**: Multi-sensor fusion models, especially those performing on large-scale benchmarks (e.g., nuScenes), often require extensive computation, which limits their practical application in real-time autonomous systems.   
**Solution by BEVFusion**: BEVFusion offers significant **computational savings**, achieving **1.9× lower computation cost** compared to point-level fusion methods. It also speeds up the view transformation (camera-to-BEV projection) with specialized pooling techniques like **interval reduction** and **precomputation**, resulting in **faster and more scalable** performance.   
### **Lack Of Flexibility Across Different 3D Perception Tasks**
**Problem**: Many existing sensor fusion methods are **task-specific**, focusing on tasks like 3D object detection without being easily adaptable to others (e.g., BEV map segmentation).   
**Solution by BEVFusion**: The framework is **task-agnostic**, meaning that it can seamlessly support different types of 3D perception tasks (e.g., 3D object detection, BEV map segmentation) with little to no architectural changes. This flexibility makes it a versatile solution for autonomous driving.   
## Key Features
### Только камерная детекция

Благодаря проекции камерных признаков на плоскость BEV независимо от лидара можно выполнять задачу детекции и сегментации только по камерам (см. раздел ниже).    

## Метрики
### Nuscenes test set

![[attachments/image_8.png]]

### Работа ночью и другие сложные погодные условия

Относительно только-лидарного CenterPoint улучшение во всех сценариях   
Почему CenterPoint ночью работает хуже? Просто более сложная подвыборка попалась? 

![image.png](attachments/cd0d1392fce2fef228267f80c87d6f03.png)

## Работа в отсутствии лидара или камер, другие зависимости

Камеры сильно улучшают метрики детекции, но на камерах тоже работает.   
![[attachments/image_f.png]]

Не понятно с какого датасета метрики   
