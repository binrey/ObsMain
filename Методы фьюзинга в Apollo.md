---
tags:
  - Apollo
  - Object_Detection/3D
  - Robotics/Perception
type: page
---

## Введение

Объединение данных в модуле восприятия apollo происходит на уровне 3Д-объектов, полученных как с камерных детекторов, так и с лидарных детекторов.   

![](attachments/73a30f0b64c68572653c264111ac5750.png)    
Ниже рассмотрены основные подходы к получению 3Д-объектов с камер   

## Camera Detection Single Stage

Модели моно-3D: 

> SMOKE - одна из первых End2End моделей для моно-3D детекции   

![](attachments/532d79b686091bbe847f56d0d2060d2b.png)    
![](attachments/5d3159b711e015f1b3411911dc4921e6.png)    

> CaDDN - учится предсказывать дискретную глубину, конвертация признаков в BEV, использование любого 3D-детектора   

![](attachments/6475b7e9b79c987dddc9e83971353ccb.png)    
![](attachments/74ed46d789e4f75906304775dc067310.png)    

## Camers Detection Multi Stage

Модель моно-3D-2D на основе yolo с дополнительными возможностями по детекции сигналов автомобилей.    

## Camera Detection BEV

> PETR - мультикамерный моно-3D детектор с трансформерным декодером   

![](attachments/8647c0d99e92f07ded6fce3a3b4b492e.png)    
