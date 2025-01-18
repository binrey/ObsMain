---
tags:
  - Sensor_Fusion
  - Object_Detection/3D
type: paper
published: 2022
source: 
fps: 6.4
inferModality:
  - L
  - LC
---
---


![2203.11496v1.pdf](attachments/2203-11496v1.pdf)    
![Screenshot from 2024-10-09 15-18-59.png](attachments/27b6d59882969e05629f59ed4f1f51ad.png)    
## Problems and Solutions   
### Difficulty handling low-quality images   
**Problem**: The performance of existing fusion methods degrades significantly when image quality is low, especially in poor lighting conditions.   
*From the paper*: "Their performance degrades seriously with low-quality image features, e.g., images in bad illumination conditions."   
**Solution**: The TransFusion model uses a soft-association mechanism that adaptively determines where and what information to take from images, making it more robust to low-quality images.   
*From the paper*: "Our fusion module... is more robust to inferior image conditions since the association between LiDAR points and image pixels is established in a soft and adaptive way."   
**Comment**: This means the model doesn't depend on perfect images to detect objects; it intelligently chooses what image data is helpful even if the images are poor.   
### Calibration errors between LiDAR and cameras   
**Problem**: LiDAR-camera fusion methods often fail when the sensors are misaligned due to errors in calibration, which affects performance.   
*From the paper*: "Finding the hard association between sparse LiDAR points and dense image pixels... heavily relies on high-quality calibration between two sensors, which is usually hard to acquire due to the inherent spatial-temporal misalignment."   
**Solution**: TransFusion solves this issue by shifting from a hard-association approach to a soft-association process, making the model less sensitive to sensor calibration errors.   
*From the paper*: "Our key idea is to reposition the focus of the fusion process, from hard-association to soft-association, leading to robustness against… sensor misalignment."   
**Comment**: This means the model can still work effectively even if the sensors aren't perfectly aligned, which is a common issue in real-world environments.   
## Key Features   
### Обогощение лидарных BEV-признаков с помощью изображений   
В начале определяется изображение с которого нужно взять информацию (на котором находится текущий 3Д-объект со своим query).    
*From the paper*: "using previous predictions as well as the calibration matrices"   
Запросы из лидарной карты вида сверху берут признаки из изображения. Но напрямую это сделать нельзя - пространство признаков с лидара не совпадает с таковым из изображения, нет соотвествия между BEV-признаками и признаками на плоскости изображения. Для решения этой задачи используется дополнительная карта, полученная с использованием матрицы проекции лидара на камеру. Эта карта поэлементно перемножается с кросс-вниманием всех голов. Таким образом каждый объект из BEV-карты "смотрит" на зону вокруг нужной области на изображении.   
![fb2f31ae27f6688e6f297f5e1e893179.png](attachments/fb2f31ae27f6688e6f297f5e1e893179.png)    
### Генерация новых запросов для BEV-карты с помощью изображений   
Для повышения полноты детекции в зонах с разреженным лидарным сигналом.   
### Только камерная детекция   
В данной модели не выполнима, поскольку весь процесс фьюзинга лидарно-центричен, BEV-карта произодит запросы к нужной камере и ее релевантным признакам.    
 --- 
## Метрики   
### Nuscenes test set   
![image.png](attachments/49e7dfefebac22e4714446bdb0a6dea2.png)    
Не по всем классам есть улучшение за счет фьюзинга!   
### Работа ночью   
![Screenshot from 2024-10-11 13-30-20.png](attachments/screenshot-from-2024-10-11-13-30-20.png)    
**СС** и **PA** - простые методы фьюзинга путем объединения признаков с камер поточечно.   
В итоге transFusion вытаскивает больше релевантной информации из ночных изображений   
Почему только-лидарный TransFusion-L ночью работает хуже? Возможно, просто выборки были маленькие и падение метрик результат случайной подвыборки. В статье комментария нет.   
### Работа при отсутствии камер   
![image.png](attachments/49e7dfefebac22e4714446bdb0a6dea2.png)    
Модель достаточно устойчива к потере данных с камер (видимо детектит в основном по точкам)   
### Детекция ближних и дальних объектов   
![image.png](attachments/image_7.png)    
## Реализации   
   
