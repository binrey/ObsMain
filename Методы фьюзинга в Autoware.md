---
tags:
  - Robotics/Perception
  - Sensor_Fusion
  - Object_Detection/3D
type: page
---

---

## Detected Objeсt Fusion

Классы 3Д-объектов с низкой уверенностью заменяются на классы объектов из 2Д-детектора.   

1. If the `existence\_probability` of a detected object is greater than the threshold, it is accepted without any further processing and published in `output`.   
2. The remaining detected objects are projected onto image planes, and if the resulting ROIs overlap with the ones from the 2D detector, they are published as fused objects in `output`. The Intersection over Union (IoU) is used to determine if there are overlaps between the detections from `input` and the ROIs from `input/rois`.   
   
**Потенциальное улучшение 3D-детекции:**   
- Уточнение класса сложных объектов (удаленных или сильно перекрытых). Потенциально ведет к уменьшению ложноотрицательных ошибок, так как верифицируются низковероятные объекты, которые иначе были бы отброшены   

[roi_detected_object_fusion](https://github.com/autowarefoundation/autoware.universe/blob/main/perception/autoware_image_projection_based_fusion/docs/roi-detected-object-fusion.md)

## Cluster Fusion

Метод похож на предыдущий, но вместо 3Д-объктов, полученных от 3Д-детекторы, используются кластеры точек (полученные, например после отсечения земли). Такие кластеры часто содержать много ложно-положительных неопределенных объектов, либо объектов, не представляющих интерес. Такие кластеры фильтруются на основе сопоставления с задетектированными с помощью камеры объектами.    
The clusters are projected onto image planes, and then if the ROIs of clusters and ROIs by a detector are overlapped, the labels of clusters are overwritten with that of ROIs by detector. Intersection over Union (IoU) is used to determine if there are overlaps between them.   
![](attachments/998531c707919086406cc1a0e5a7030b.png)    
**Потенциальное улучшение 3D-детекции:**   

- Уточнение класса неопределенных детекций   
- Уменьшение числа ложноотрицательных ошибок путем дополнением детекций кластерами, с классом от 2Д-модели   

[roi_cluster_fusion](https://github.com/autowarefoundation/autoware.universe/blob/main/perception/autoware_image_projection_based_fusion/docs/roi-cluster-fusion.md)    

## Point Cloud Fusion

Создание новых 3Д-объектов в виде кластеров точек на основе 2Д-детекций (рамки, полученные 2Д-детекторои проецируются на облако точек). Особенно актуально для маленьких отдельно стоящих объектов, чья ориентация не важна. В целом делает тоже самое, что предыдущий метод, но на сыром облаке точек, без предварительной кластеризации.   

1. The pointclouds are projected onto image planes and extracted as cluster if they are inside the unknown labeled ROIs.   
2. Since the cluster might contain unrelated points from background, then a refinement step is added to filter the background pointcloud by distance to camera.   

![](attachments/a080bd868320afd2a68b423da59cc799.png)    

**Потенциальное улучшение 3D-детекции:**   
- Уменьшение числа ложноотрицательных ошибок путем дополнением детекций кластерами, с классом от 2Д-модели   

[roi_pointcloud_fusion](https://github.com/autowarefoundation/autoware.universe/blob/main/perception/autoware_image_projection_based_fusion/docs/roi-pointcloud-fusion.md)    

## Segmentation Pointcloud

Удаление из лидарного облака точек, которые не относятся к объектам. Класс точек определяется 2Д-сегментационной моделью, чьи маски проецируются на облако точек. В визуализации autoware автомобили почему-то обведены как детекторные рамки, а не маски.    

1. The pointclouds are projected onto the semantic/instance segmentation mask image which is the output from 2D segmentation neural network.   
2. The pointclouds are belong to segment such as road, sidewalk, building, vegetation, sky ... are considered as less important points for autonomous driving and could be removed.   

![](attachments/9acc124f7afed7591458d11fdcb25444.png)    

**Потенциальное улучшение 3D-детекции:**   
- Уменьшение числа ложноположительных ошибок путем фильтрации точек, не содержащих объекты по мению 2Д-модели   

[segmentation_pointcloud_fusion](https://github.com/autowarefoundation/autoware.universe/blob/main/perception/autoware_image_projection_based_fusion/docs/segmentation-pointcloud-fusion.md)    

## Pointpainting Fusion

Подход [[PointPainting. Sequential Fusion for 3D Object Detection]] при котором облако точек "расскрашивается" классами из 2Д-детекции или сегментации. Далее идет обработка стандартным 3Д-детектором.   

The lidar points are projected onto the output of an image-only 2d object detection network and the class scores are appended to each point. The painted point cloud can then be fed to the centerpoint network.   

![](attachments/1dc485396212097c97bc08866caf61cd.jpg)    
**Потенциальное улучшение 3D-детекции:**   

- Уменьшение числа ложноотрицательных ошибок путем добавления информации о классе от 2Д-модели (особенно полезно в зонах, где лидарный сигнал разреженный)   
- Уменьшение числа ложноположительных ошибок путем подсвечивания 2Д-моделью, что в данной области объекта нет   

[pointpainting_fusion](https://github.com/autowarefoundation/autoware.universe/blob/main/perception/autoware_image_projection_based_fusion/docs/pointpainting-fusion.md)    

## Сравнительная таблица

|               **Метод** |                                        **Типы объектов** |                         **Ложно-положительные ошибки** |                         **Ложно-отрицательные ошибки** |
|:------------------------|:---------------------------------------------------------|:-------------------------------------------------------|:-------------------------------------------------------|
|   Detected Objet Fusion |                                3D-детекции + 2D-детекции |                                                        |                                                      ✔ |
|          Cluster Fusion |                                3D-кластеры + 2D-детекции |                                                        |                                                      ✔ |
|      Point Cloud Fusion |                                   3D-точки + 2D-детекции |                                                        |                                                      ✔ |
| Segmentation Pointcloud |                                      3D-точки + 2D-маски |                                                      ✔ |                                                        |
|    Pointpainting Fusion |                      3D-точки + 2D-маски или 2D-детекции |                                                      ✔ |                                                      ✔ |
## Полезные ссылки
[Autoware sensor fusion related content](https://blog.csdn.net/weixin_43493439/article/details/124063044)   
[Про image_projection_based_fusion](https://blog.csdn.net/weixin_45432823/article/details/140067274)
