## Pointpainting
Обучение центерпоинта на точках, аугментиравнных классами от сегментации кадра

**Плюсы**
-  Максимально понятная логика
- Обучение 2д-модели на задаче, напрямую влияющей на итоговый результат
- Получение сегментации облака точек «в подарок»
- Проекция из 3д в 2д менее чувствительна к ошибкам определения дистанции для удаленных объектов
- Модель может работать только на лидарных данных
- Можно легко запуститься с несколькими камерами - нужно спроецировать точки на каждую камеру с помощью своих матриц трансформации и просто присвоить соответствующими точкам классы. На пересекающихся камерах можно выбирать класс с большей уверенностью 

**Минусы**
- Улучшение 2д-модели потребует разметки сегментации
- Необходимость запускать две отдельные модели
- Не работает без лидарных данных
- Полноценное обучение потребует скалиброванных размеченных 2д и 3д кадров
- Использует только 5% 2д-признаков
## BEVFusion
Метод проекции данных в единое Bev-представление, затем детекция декодером центерпоинта

**Плюсы**
- Обучение модели вообще не требует 2д-разметки
- Единая архитектура end-to-end
- Модель может работать только на камерах
- Модель может работать только на лидарных данных
- Быстрее PointPainting в 1.6 раз
- Показывает более высокие метрики чем PointPainting на 3 п.п. [[attachments/image_8.png]]
- Использует 2д-признаки на 100%
- Можно легко запуститься с несколькими камерами - нужно спроецировать карты признаков каждой камеры в соответствующую зона единого пространства BEV. На пересекающихся зонах можно усреднить значения.

**Минусы**
- Изображение сжимается в пространство признаков, которые сложнее интерпретировать
- Обучение 2д-модели происходит косвенным путем, только на приложенных к облакам точек синхронных кадрах, что замедляет время релиза
- Польза от предобучения 2д-энкодера на наших данных кажется очень ограниченной - достаточно текущих данных чисто для ускорения сходимости, не для повышения финальных метрик
- Нельзя обучать 2д-энкодер на отдельной задаче детекции - нужны расстояния до объектов или карта глубины