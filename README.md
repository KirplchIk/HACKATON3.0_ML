Отлично! Вот пример полной документации для вашего проекта `person_detector_hog_svm`, оформленной в виде файла `README.md`. Вы можете скопировать это в файл `README.md` в корне вашего проекта.

```markdown
# Детектор Людей на Основе HOG + SVM (person_detector_hog_svm)

## Описание

Этот проект реализует классический пайплайн для обнаружения людей на изображениях с использованием метода Гистограмм Ориентированных Градиентов (HOG) для извлечения признаков и Линейного Метода Опорных Векторов (Linear SVM) для классификации. Проект включает этапы подготовки данных, обучения модели, детекции и оценки, а также предоставляет простое API на FastAPI для использования обученного детектора.

Основная цель - демонстрация полного цикла создания детектора объектов с использованием традиционных подходов компьютерного зрения и машинного обучения. Может быть использован как база для задач подсчета людей (например, на остановках, в помещениях) с пониманием ограничений данного метода.

## Основные Возможности

*   **Подготовка данных:** Извлечение позитивных (люди) и негативных (фон) сэмплов из датасета в формате YOLO.
*   **Извлечение признаков:** Вычисление HOG-признаков для подготовленных сэмплов.
*   **Обучение модели:** Тренировка классификатора Linear SVM на HOG-признаках.
*   **Детекция:** Обнаружение людей на новых изображениях с использованием скользящего окна и пирамиды масштабов.
*   **Оценка:** Расчет метрики Mean Absolute Error (MAE) для оценки качества подсчета людей на тестовом наборе данных.
*   **API:** Веб-интерфейс на FastAPI для детекции людей на загружаемых изображениях.

## Структура Проекта

```
person_detector_hog_svm/
│
├── data/                     # Папка для хранения датасета (не включена в репозиторий)
│   └── person-detection-10k/ # Пример ожидаемой структуры датасета
│       ├── train/
│       │   ├── images/
│       │   └── labels/       # Аннотации в формате YOLO (.txt)
│       ├── val/
│       │   ├── images/
│       │   └── labels/
│       └── test/
│           ├── images/
│           └── labels/
│
├── output/                   # Папка для всех генерируемых файлов (создается автоматически)
│   ├── prepared_data/        # Нарезанные позитивные/негативные сэмплы
│   │   ├── positive_samples/
│   │   └── negative_samples/
│   ├── features/             # Сохраненные HOG-признаки и метки (.npy)
│   ├── models/               # Обученная модель SVM (.joblib) и параметры HOG (.json)
│   └── detections/           # Изображения с результатами детекции с тестового набора
│
├── scripts/                  # Исполняемые скрипты для запуска этапов пайплайна
│   ├── 1_prepare_data.py     # Запуск подготовки данных
│   ├── 2_train_model.py      # Запуск извлечения признаков и обучения модели
│   └── 3_run_detection.py    # Запуск детекции на тестовом наборе и оценка MAE
│
├── src/                      # Основной исходный код проекта (модули)
│   ├── __init__.py           # Делает src пакетом Python
│   ├── config.py             # Конфигурация проекта (пути, параметры HOG/SVM/детекции)
│   ├── data_preparation.py   # Логика нарезки сэмплов
│   ├── feature_extraction.py # Логика вычисления HOG-признаков
│   ├── training.py           # Логика обучения SVM
│   ├── detection.py          # Логика детекции людей и загрузки детектора
│   └── utils.py              # Вспомогательные функции (работа с аннотациями, IoU, NMS, ресайз)
│
├── api_fastapi.py            # Реализация API на FastAPI
│
└── README.md                 # Этот файл документации
```

## Требования

*   Python 3.8+
*   Основные библиотеки (установите с помощью pip):
    *   `opencv-python`
    *   `numpy`
    *   `scikit-learn`
    *   `joblib`
    *   `tqdm`
    *   `fastapi`
    *   `uvicorn[standard]` (для запуска API)
    *   `python-multipart` (для загрузки файлов в FastAPI)
    *   `imutils` (использовался ранее для NMS, можно удалить, если `cv2.dnn.NMSBoxes` работает стабильно)
    *   `matplotlib` (опционально, для построения графика в `3_run_detection.py`)

**Установка зависимостей:**

1.  (Рекомендуется) Создайте и активируйте виртуальное окружение:
    ```bash
    python -m venv venv
    source venv/bin/activate  # Linux/macOS
    # или
    venv\Scripts\activate    # Windows
    ```
2.  Установите зависимости из файла (создайте `requirements.txt` со списком выше):
    ```bash
    pip install -r requirements.txt
    # или установите по отдельности:
    # pip install opencv-python numpy scikit-learn joblib tqdm fastapi "uvicorn[standard]" python-multipart matplotlib
    ```

## Датасет

*   Проект ожидает датасет в формате **YOLO**.
*   Каждому изображению (`.jpg`, `.png`) в папке `images/` должен соответствовать текстовый файл (`.txt`) с таким же именем в папке `labels/`.
*   Каждая строка в `.txt` файле описывает один объект:
    `<class_id> <xc_norm> <yc_norm> <width_norm> <height_norm>`
    *   `<class_id>`: Целочисленный ID класса (в `config.py` указано `PERSON_CLASS_ID = 0`).
    *   Остальные значения: Координаты центра (x, y), ширина и высота рамки, **нормализованные** относительно размеров изображения (значения от 0 до 1).
*   Предполагается стандартная структура с папками `train`, `val`, `test`. Положите ваш датасет в папку `data/` (или измените путь `DATA_DIR` в `src/config.py`).

## Конфигурация

Все основные параметры проекта настраиваются в файле `src/config.py`. Это включает:

*   **Пути:** К датасету, папкам для вывода результатов, моделям.
*   **Параметры подготовки данных:** Размер окна (`WINDOW_SIZE`), количество негативных сэмплов (`NEG_SAMPLES_PER_IMAGE`), порог IoU для негативов (`NEG_IOU_THRESHOLD`).
*   **Параметры HOG:** `winSize`, `blockSize`, `blockStride`, `cellSize`, `nbins` и другие. **Критически важно** подбирать эти параметры для хорошего качества признаков.
*   **Параметры SVM:** Параметр регуляризации `SVM_C`.
*   **Параметры детекции:** Шаг окна (`DETECTION_WINDOW_STEP`), масштаб пирамиды (`DETECTION_PYRAMID_SCALE`), порог уверенности SVM (`DETECTION_THRESHOLD`), порог NMS (`NMS_THRESHOLD`). **Требуют тюнинга** для баланса точности/полноты и скорости.

Обратитесь к комментариям в `src/config.py` за подробным описанием каждого параметра.

## Использование (Пайплайн)

Запускайте скрипты из папки `scripts/` последовательно. Убедитесь, что вы находитесь в корневой папке проекта (`person_detector_hog_svm/`) или что ваше виртуальное окружение активно.

1.  **Подготовка данных:**
    *   Этот шаг нарезает позитивные и негативные сэмплы из обучающего набора данных и сохраняет их в `output/prepared_data/`.
    *   Запустите:
        ```bash
        python scripts/1_prepare_data.py
        ```

2.  **Извлечение признаков и Обучение модели:**
    *   Этот скрипт сначала вычисляет HOG-признаки для всех нарезанных сэмплов (сохраняет в `output/features/`), а затем обучает модель Linear SVM на этих признаках (сохраняет в `output/models/`). Параметры HOG также сохраняются.
    *   Запустите:
        ```bash
        python scripts/2_train_model.py
        ```

3.  **Детекция и Оценка:**
    *   Этот скрипт загружает обученный детектор (SVM+HOG) и прогоняет его по **тестовому** набору данных.
    *   Он вычисляет метрику MAE (средняя абсолютная ошибка в подсчете людей) и сохраняет изображения с результатами детекции в `output/detections/`.
    *   Запустите:
        ```bash
        python scripts/3_run_detection.py
        ```
    *   Проанализируйте выведенное значение MAE и просмотрите сохраненные изображения для оценки качества работы детектора. При необходимости вернитесь к `config.py` для тюнинга параметров детекции (`DETECTION_THRESHOLD`, `NMS_THRESHOLD`) и перезапустите этот скрипт.

## Использование API (FastAPI)

Проект предоставляет API для детекции людей на загружаемых изображениях.

1.  **Запуск сервера API:**
    *   Убедитесь, что модель обучена (файл `output/models/svm_model.joblib` существует).
    *   Запустите сервер Uvicorn из корневой папки проекта:
        ```bash
        uvicorn api_fastapi:app --reload --host 0.0.0.0 --port 8000
        ```
        *   `--reload`: Автоматически перезапускает сервер при изменении кода (удобно для разработки).
        *   `--host 0.0.0.0`: Делает API доступным по сети (а не только с localhost).
        *   `--port 8000`: Указывает порт, на котором будет работать API.

2.  **Отправка запроса на детекцию:**
    *   API ожидает POST-запрос на эндпоинт `/predict` с файлом изображения в поле `image`.
    *   **Пример с `curl`:**
        ```bash
        curl -X POST -F "image=@/путь/к/вашему/изображению.jpg" http://localhost:8000/predict
        ```
        (Замените `/путь/к/вашему/изображению.jpg` на реальный путь). Если сервер запущен не локально, замените `localhost` на IP-адрес сервера.
    *   **Пример с Python (`requests`):**
        ```python
        import requests

        api_url = "http://localhost:8000/predict"
        image_path = "/путь/к/вашему/изображению.jpg" # Замените путь

        try:
            with open(image_path, 'rb') as f:
                files = {'image': (image_path, f, 'image/jpeg')} # Передаем имя файла, файл и mime-тип
                response = requests.post(api_url, files=files)
                response.raise_for_status() # Проверка на HTTP ошибки
                print(response.json())
        except requests.exceptions.RequestException as e:
            print(f"Ошибка при запросе к API: {e}")
        except FileNotFoundError:
            print(f"Ошибка: Файл не найден по пути {image_path}")

        ```

3.  **Ожидаемый ответ (JSON):**
    ```json
    {
      "success": true,
      "error": null,
      "person_count": 5 // Количество обнаруженных людей
      // "boxes": [[x1, y1, w1, h1], [x2, y2, w2, h2], ...] // Опционально, если добавить в API
    }
    ```
    В случае ошибки:
    ```json
    {
      "success": false,
      "error": "Описание ошибки",
      "person_count": 0
    }
    ```

4.  **Проверка состояния API:**
    *   Можно отправить GET-запрос на `/health`, чтобы проверить, загрузился ли детектор.
    *   `curl http://localhost:8000/health`

## Оценка Модели

*   Основная метрика качества для задачи подсчета - **Mean Absolute Error (MAE)**, рассчитываемая в скрипте `3_run_detection.py`. Она показывает среднее абсолютное отклонение предсказанного количества людей от истинного. Чем ниже MAE, тем лучше.
*   Визуальный анализ результатов детекции (изображений в `output/detections/`) крайне важен для понимания типов ошибок модели (ложные срабатывания, пропуски).

## Ограничения

*   **HOG+SVM** - это классический, но устаревший подход по сравнению с современными **детекторами на основе глубокого обучения** (YOLO, SSD, Faster R-CNN и т.д.).
*   **Точность:** Ожидаемая точность обнаружения и подсчета будет ниже, чем у нейросетевых моделей, особенно в сложных условиях.
*   **Устойчивость:** Метод чувствителен к перекрытиям объектов, изменениям освещения, нестандартным позам, вариациям масштаба.
*   **Скорость:** Детекция с использованием скользящего окна и пирамиды масштабов может быть медленной, особенно на больших изображениях.
*   **Ложные срабатывания:** При низком пороге уверенности возможны срабатывания на текстурах фона, похожих на людей.

## Возможные Улучшения

*   **Hard Negative Mining:** Вместо случайного выбора негативных сэмплов, можно добавить этап поиска "сложных" негативов (тех, на которых текущий детектор ошибается) и дообучить модель на них.
*   **Тюнинг Гиперпараметров:** Систематический подбор параметров HOG, SVM (`C`) и детекции (`DETECTION_THRESHOLD`, `NMS_THRESHOLD`) с использованием валидационного набора данных (например, с помощью GridSearch или Optuna).
*   **Использование Других Классификаторов:** Попробовать другие классификаторы поверх HOG-признаков (например, Random Forest, Gradient Boosting).
*   **Переход на Глубокое Обучение:** Для существенного повышения качества рассмотреть использование предобученных нейросетевых детекторов (YOLOv5/v7/v8, Faster R-CNN и т.д.).
*   **Улучшенный NMS:** Исследовать другие варианты NMS (например, Soft-NMS), особенно для плотных скоплений людей.
*   **Дополнительные Метрики:** Рассчитать стандартные метрики детекции объектов (Precision, Recall, F1, mAP), если важна точность рамок, а не только подсчет.

## Лицензия

(Укажите здесь лицензию, если она есть, например: MIT License)
```
