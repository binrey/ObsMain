---
tags:
  - Object_Detection/3D
  - Robotics/Perception
type: video
source: https://www.youtube.com/watch?v=wMHfN1mzwME&list=PL3N9otbGBVLc3jdm6yrPtCWdE7C8AsNAy&index=9
---

---

## Camera-Based 3D Perception

One of the core topics of Dr. Alvarez’s talk is how cameras are used to perceive the 3D environment around the vehicle. Traditionally, autonomous systems have relied heavily on LiDAR to capture 3D data. However, cameras are a more cost-effective solution, though they introduce the challenge of accurately transforming 2D images into 3D space. This process involves using image encoders, followed by transformations to understand the depth and the spatial layout of the environment.   
A major challenge here is balancing accuracy and efficiency. Larger models can offer higher accuracy but are computationally expensive and difficult to deploy in real-time, especially in resource-constrained environments like cars. Nvidia addresses this challenge by developing deep learning models that can perform 2D-to-3D transformations efficiently, using techniques like forward projection (mapping 2D points to 3D space) and backward projection (working backward from 3D points to 2D images). These techniques help improve the performance and scalability of perception models.   

![[attachments/Pasted image 20250117100736.png]]

BEV-детекция: нужно проецировать кадр на поверхность -> нужно получить глубину

![[attachments/Pasted image 20250121003510.png]]

## The Importance of Data Efficiency
### Качество важнее размера?
Устоявшийся подход - больше данных всегда улучшают качество модели. Авторы показывают (что важно, на примере задачи автономного вождения) что правильно отобранные 80% лучше чем 100%: 

![[attachments/Pasted image 20250121003928.png]]

### Как оценить сколько данных нужно
Можно предсказать сколько нужно данных для достижения требуемого значения метрики:

![[attachments/Pasted image 20250121005143.png]]

Можно предсказывать сколько нужно данных, чтобы обучиться на новом классе с заданной точностью:

![[attachments/Screenshot 2025-01-21 at 00.57.55.png]]

Весть процесс активного обучения:

![[attachments/Screenshot 2025-01-21 at 01.02.52.png]]

Вопросы: 

- как определить что новые собранные данные будут полезны для модели?
- как за (желательно за один проход) по сети узнать ее неопределенность в ответе на данных. Она должна быть откалибрована.
![[attachments/Screenshot 2025-01-21 at 01.15.11.png]]

## Улучшение детекции дальних 3Д-объектов:

![[attachments/Pasted image 20250121012137.png]]

[Yang_Improving_Distant_3D_Object_Detection_Using_2D_Box_Supervision_CVPR_2024_paper.pdf](https://openaccess.thecvf.com/content/CVPR2024/papers/Yang_Improving_Distant_3D_Object_Detection_Using_2D_Box_Supervision_CVPR_2024_paper.pdf)

## Редкие объекты

![[attachments/Pasted image 20250121013509.png]]

## Улучшение модели при увеличении данных но фиксированном бюджете:

![[attachments/Pasted image 20250121013712.png]]

## Network Robustness: A Critical Factor

Another key focus in the research is the pruning of deep learning models. Autonomous driving systems require models that are highly accurate but also small and efficient enough to run in real-time on hardware embedded in vehicles. Model pruning is a technique that compresses large models without losing much accuracy. By removing less important parts of the neural network, Nvidia is able to produce models that retain robustness but are more compact and suitable for deployment in edge devices, such as those used in cars.   
Dr. Alvarez describes how large neural networks are typically very robust, but when compressed into smaller models, their robustness often diminishes. Nvidia tackles this by combining pruning with distillation (a method where a smaller model is trained to mimic a larger, more accurate model), thus maintaining performance while reducing model size. 

![[attachments/Pasted image 20250121014015.png]]

3Д-детекция особенно чувствительна к примерам вне обучающего распределения:

![[attachments/Pasted image 20250121014053.png]]


In real-world driving scenarios, a model’s robustness is vital. Dr. Alvarez discusses how small variations in the environment—such as changes in lighting, weather, or sensor noise—can significantly affect the performance of autonomous driving models. Nvidia addresses this by designing models that are more robust to such variations, using attention mechanisms to improve the stability of the perception pipeline.   
Self-attention modules are particularly useful in making networks more resilient to disturbances in input data. By improving the self-attention mechanisms and developing fully attention-based networks, Nvidia has been able to enhance the robustness of its models, which is crucial for ensuring the safety and reliability of autonomous vehicles in diverse environments.

Самовнимание показывает отличные результаты в вопросе устойчивости к нарушению сигнала с сенсора: падение метрик всего на 20%. 

![[attachments/Screenshot 2025-01-21 at 08.26.11 1.png]]

Как избавиться от падения совсем? Нужно больше самовнимания в блоках модели.

![[attachments/Screenshot 2025-01-21 at 08.31.30.png]]

Такой подход делает модель еще более устойчивой!

![[attachments/Screenshot 2025-01-21 at 08.31.54.png]]
Текущие модели не устойчивы к смене камеры, угла обзора, высоты установки и т.д. Выход - давайте моделировать генеративными моделями небольшие вариации перечисленных выше параметров и учиться на этих данных.

![[attachments/Pasted image 20250121084102.png]]
## Model Pruning and Compactness
Но предыдущие результаты справедливы для больших моделей. Маленькие - не достаточно устойчивы. Как эффективно перенести свойство устойчивости с большой модели на модель реального времени? Вместо подбора архитектуры малой модели лучше прунить большую - так сохраняется ее устойчивость:

![[attachments/Pasted image 20250121083956.png]]

![[attachments/Screenshot 2025-01-21 at 08.40.03.png]]

## Conclusion

The development of autonomous vehicle perception systems is a complex task that involves many trade-offs between data collection, model efficiency, and network robustness. Dr. Jose Alvarez’s work at Nvidia demonstrates how cutting-edge techniques like 2D-to-3D transformations, model pruning, and attention mechanisms can make significant strides in advancing the field. The focus on selecting the right data, improving efficiency, and ensuring robust performance in the face of real-world challenges will continue to shape the future of autonomous driving technology.   
![[attachments/Pasted image 20250121084344.png]]
