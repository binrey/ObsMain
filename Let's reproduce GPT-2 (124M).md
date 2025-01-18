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
- GPT-2 использует pre-norm, что улучшает проход градиентов за счет того, что всегда остается оригинальная ветвь с потоком градиентов (см. x = x + …)   

```
class Block(nn.Module):

    def __init__(self, config):
        super().__init__()
        self.ln_1 = nn.LayerNorm(config.n_embd)
        self.attn = CausalSelfAttention(config)
        self.ln_2 = nn.LayerNorm(config.n_embd)
        self.mlp = MLP(config)

    def forward(self, x):
        x = x + self.attn(self.ln_1(x))
        x = x + self.mlp(self.ln_2(x))
        return x
```

## Attention-MLP
- Слой внимания - операция взвешенной суммы, место где токены "коммуницируют" друг с другом, операция reduce
- MLP обрабатывает токены независимо друг от друга, что-то вроде операции map   
- Таким образом, можно провести аналогию между [map-reduce](https://en.wikipedia.org/wiki/MapReduce) (распределенные вычисления) и трансформерными последовательностями слоёв attention-MLP-attention-MLP-...

## Оценка loss

Случайно инициализированная модель предсказывает каждый токен равновероятным, тогда можно оценить функцию потерь как:   

```
loss = -ln(1/vocab_size)
```

## Что предсказывает GPT

Трансформер предсказывает не совсем вероятность следующего слова. Размер выходного тензора - (B, T, vocab_size), он же подается в кроссэнтропию и сравнивается с таким же тензором, состоящим из токенов, сдвинутых на один в будущее. То есть технически сеть предсказывает целый батч последовательностей токенов - тензор размером, равным исходному, но представляющий будущие токены. Все из них уже были видны в исходных последовательностях, кроме последних токенов, их и берем.    
Похоже на режим sequence to sequence в рекурентных сетях. Можно предсказывать не только следующий токен, а делать, например, сегментацию текста.   

## Fused опция для AdamW

Опция актуальна при вычислениях на видеокарте (cuda или mps тоже?) и позволяет ускорить обновление параметров сети путем замены итеративного обновления на матричное, несколько параметров за раз на ускорителе.

> [!NOTE] AdamW docs
> The foreach and fused implementations are typically faster than the for-loop, single-tensor implementation, with fused being theoretically fastest with both vertical and horizontal fusion. As such, if the user has not specified either flag (i.e., when foreach = fused = None), we will attempt defaulting to the foreach implementation when the tensors are all on CUDA. Why not fused? Since the fused implementation is relatively new, we want to give it sufficient bake-in time. To specify fused, pass True for fused. To force running the for-loop implementation, pass False for either foreach or fused.

## Правильные и неправильные числа

Правильные числа в ML - кратные 2, в идеале 2 в n-степени, или в целом, чем больше делений на 2 можно выполнить тем лучше. Например, увеличение размера словаря с "плохого" 50257 до четного размера в 50304 несмотря на увеличение по факту числа операций, тем не менее ускоряет обучение.

## Проверка в Colab

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