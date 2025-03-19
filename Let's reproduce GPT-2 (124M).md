---
tags:
  - LLM
type: video
source: https://www.youtube.com/watch?v=l8pRSuU81PU&list=PLtra0TFhKk-TZgC_IRobYBIriBIakzBuc
тема:
---

---

## Нормализация

- Операция сложения разделяет градиент на две равные ветви   
- В оригинальном трансформере использовалась post-norm нормализация
- GPT-2 использует pre-norm, который улучшает проход градиентов за счет того, что всегда остается оригинальная ветвь с не модифицированным потоком градиентов (см. x = x + …)   

```
    def forward(self, x):
        # Self-Attention sublayer
        attn_output, _ = self.self_attn(x, x, x)
        x = self.norm1(x + attn_output)  # Post-Norm

        # Feed-forward sublayer
        linear_output = self.linear(x)
        x = self.norm2(x + linear_output)  # Post-Norm
        return x
        
    def forward(self, x):
        # Self-Attention sublayer
        attn_output, _ = self.self_attn(self.norm1(x), self.norm1(x), self.norm1(x))  # Pre-Norm
        x = x + attn_output

        # Feed-forward sublayer
        linear_output = self.linear(self.norm2(x))  # Pre-Norm
        x = x + linear_output
        return x
```

## Attention-MLP
- Слой внимания - операция взвешенной суммы, место где токены "коммуницируют", взаимодействуют друг с другом. Очень похоже на операцию **Reduce** в MapReduce процедуре.
- MLP обрабатывает токены независимо друг от друга, они не обмениваются информацией друг с другом, таким образом, это что-то вроде операции map в MapReduce процедуре.  
- Можно провести аналогию между [map-reduce](https://en.wikipedia.org/wiki/MapReduce) (распределенные вычисления) и трансформерными последовательностями слоёв attention -> MLP -> attention -> MLP... 
- Для меня такое сравнение осталось не до конца понятным, ведь линейный слой MLP это как раз тоже взвешенная сумма нейронов. И по сути обмен информацией. 
## Оценка loss

Случайно инициализированная модель предсказывает каждый токен равновероятным, тогда можно оценить функцию потерь как:   

```
loss = -ln(1/vocab_size)
```

## Что предсказывает GPT

Обычно говорят, что трансформер предсказывает вероятность следующего слова. На самом деле не только его. Размер выходного тензора - (B, T, vocab_size), в функции потерь он сравнивается с тензором такого же размера, состоящим из токенов, сдвинутых на один в будущее. То есть технически сеть предсказывает батч последовательностей токенов - тензор размером, равным исходному, но представляющий будущие токены каждой последовательности. Все из них уже были видны во входных предложениях, кроме самого последнего токена, который предсказала модель. Эти последние токены на каждой итерации составят ответ модели. 

## Fused опция для AdamW

Опция актуальна при вычислениях на видеокарте (cuda) и позволяет ускорить обновление параметров сети путем замены итеративного обновления на матричное (несколько параметров за раз на ускорителе).

> [!NOTE] AdamW docs
> The foreach and fused implementations are typically faster than the for-loop, single-tensor implementation, with fused being theoretically fastest with both vertical and horizontal fusion. As such, if the user has not specified either flag (i.e., when foreach = fused = None), we will attempt defaulting to the foreach implementation when the tensors are all on CUDA. Why not fused? Since the fused implementation is relatively new, we want to give it sufficient bake-in time. To specify fused, pass True for fused. To force running the for-loop implementation, pass False for either foreach or fused.

## Правильные и неправильные числа

Правильные числа в ML - кратные 2 в n-степени, чем больше n, тем лучше. Например, увеличение размера словаря с "плохого" 50257 до четного размера в 50304 несмотря на увеличение по факту числа операций, тем не менее ускоряет обучение!

## Проверка в Colab
Мой блокнот для запуска кода лекции в колабе. Можно бесплатно проверить все трюки, работающие для одной видеокарты. Ниже в таблице результаты проверки. Они действительно работают! Правда, скорее всего будет значительный разброс при различных запусках :(

```embed
title: "Google Colab"
image: "https://colab.research.google.com/img/colab_favicon_256px.png"
description: "Sign in"
url: "https://colab.research.google.com/drive/1fEEzkt5owZSej6MPuUY-ZCRD6Z12UTqF?authuser=0#scrollTo=W7u6B1TLRsla&uniqifier=1"
```

| element                                                  | time, sec | GPU mem |
| :------------------------------------------------------- | --------: | ------: |
| autocast + fine vocab_size + fused AdamW + torch.compile | 22.3      | 8.2     |
| autocast + fine vocab_size + fused AdamW                 | 25.8      | 11.2    |
| autocast + fine vocab_size                               | 25.8      | 11.2    |
| autocast                                                 | 31.1      | 11.2    |
| -                                                        | 128.2     | 14.5    |