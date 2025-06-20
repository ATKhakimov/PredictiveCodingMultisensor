# Восстановление мультимодальных сигналов с помощью импульсного предиктивного кодирования

Этот репозиторий содержит реализацию и эксперименты, описанные в работах:
- «Восстановление сигнала с помощью импульсного мультисенсорного предиктивного кодирования» (тезисы)
- «Сравнение моделей и методов предиктивного кодирования в задаче восстановления мультимодальных сигналов» (статья в журнале «Микроэлектроника»)

Основная цель — разработка биологически правдоподобной модели **импульсной нейронной сети (ИмНС)**, способной восстанавливать зрительные и аудиальные сигналы на основе **предиктивного кодирования** и **локального обучения**. Архитектура использует нейроны **leaky integrate-and-fire (LIF)** и реализует модифицированное **хеббовское обучение с распадом**.

## Основная концепция

Модель обучается предсказывать входные сенсорные данные и корректировать прогнозы на основе локальной ошибки восстановления. Это позволяет:
- объединять визуальные и аудиальные модальности в общем скрытом слое;
- отказаться от глобального градиента;
- использовать энергоэффективную событийную обработку (спайки).

## Содержимое репозитория

### `Predictive coding Multisensor_сpu_v2.ipynb`  
Главный ноутбук с реализацией модели, описанной в тезисах и статье. Включает:
- обновление весов с помощью модифицированного правила Хебба с распадом
- пространственное кодирование: 784 входных нейрона (изображение MNIST) + 5000 нейронов (аудио FSDD);
- скрытый слой: 370 LIF-нейронов;
- симметричная архитектура прямых и обратных синапсов (feedforward/feedback);
- предиктивная схема обновления весов;
- визуализация восстановления изображений и аудиосигналов;
- оценка скрытого слоя через логистическую регрессию и сверточные классификаторы.

### `PredictiveCoding_GPU.ipynb`  
Расширенная реализация предиктивного кодирования:
- перенос модели на **GPU** с использованием библиотеки `Brian2CUDA`;
- работа с **расширенным датасетом**: больше пар «изображение + аудио» и более длительные симуляции;
- добавлен **детализированный мониторинг активности**: логирование потенциалов, плотности спайков, восстановления по модальностям, визуализация предсказаний и ошибок;
- поддержка батчевого режима и ускоренной обработки на видеокартах;
- предназначен для оценки масштабируемости модели и анализа скрытой динамики при увеличенной нагрузке.

### `MPC_comparison.ipynb`  
Сравнение разных типов автоэнкодеров в задаче восстановления сигнала:
- **Классический автоэнкодер (MLP)**, обучаемый градиентным методом;
- **Спайковый предиктивный автоэнкодер** на основе LIF-нейронов и локального обучения;
- исследуется **динамика обучения** (ошибка по эпохам), **стабильность сходимости**, **точность реконструкции**;
- проводится визуальное сравнение качества восстановления изображений;
- обсуждаются **преимущества и ограничения событийного подхода**.


### `MPC_CNN.ipynb`  
Реализация **мультимодальной сверточной нейросети (CNN)** без использования спайкового кодирования:
- модель принимает **изображение (через Conv2D)** и **аудиосигнал (через Conv1D)** в двух параллельных ветках;
- признаки объединяются и подаются в полносвязный слой;
- обучается как эталонная архитектура **end-to-end**, без скрытого слоя предиктивной SNN;
- используется для **сравнительной оценки качества латентных признаков** по точности классификации;
- показывает ограниченную устойчивость и склонность к переобучению при малом объеме данных.
- 
##  Датасеты

- **MNIST**: изображения 28×28, 10 классов (0–9)
- **FSDD** (Free Spoken Digit Dataset): аудиозаписи произнесения цифр, синхронизированные с изображениями

##  Основные параметры модели (для пространственного кодирования)

| Параметр              | Значение                |
|-----------------------|--------------------------|
| Входной слой          | 784 + 5000 нейронов      |
| Скрытый слой          | 370 нейронов             |
| Окно симуляции        | 100 мс                   |
| Постоянная времени τ  | 10 мс                    |
| Скорость обучения η   | 1e-6                     |
| Распад λ              | 4e-3                     |

##  Сводные метрики

###  Качество восстановления входного сигнала

| Модель | Метрика | Сигнал | Значение     |
|--------|---------|--------|--------------|
| PC     | MSE     | IMG    | **0.02–0.04** |
| PC     | COR     | IMG    | **0.96–0.98** |
| PC     | MSE     | AUDIO  | **0.07–0.09** |
| PC     | COR     | AUDIO  | **0.88–0.91** |

###  Оценка качества скрытого слоя

| Модель       | Метрика  | Сигнал        | Значение     |
|--------------|----------|---------------|--------------|
| PC (hidden)  | Accuracy | IMG + AUDIO   | **0.74–0.79** |
| LogReg       | Accuracy | IMG           | **0.90–0.91** |
| CNN (hidden) | Accuracy | IMG + AUDIO   | **0.70–0.72** |

> **Таблица:** Сравнение качества реконструкции мультимодальной предиктивной SNN (PC) и точности классификации скрытого слоя в сравнении с классическими моделями.


##  Больше информации

Подробности описаны в статьях:
- [Тезисы (DOCX)](./Тезисы_Хакимов_мфти.docx)
- [Научная статья в журнале «Микроэлектроника» (DOCX)](./Статья_Микроэлектроника_2025_Хакимов_итог.docx)

##  Контакт

Артём Хакимов  
Email: khakimov.at@phystech.edu  
МФТИ
