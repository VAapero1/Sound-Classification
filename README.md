# Sound-Classification

<a href="https://ibb.co/wdj1v2H"><img src="https://i.ibb.co/fXL7cTR/3-D-spectrogram.png" alt="3-D-spectrogram" border="0"></a>

# Часть 1
**Загрузка файлов. Составные части звука с последующим извлечением признаков.**

---

Для начала советую вам перейти по [ссылке](https://www.kaggle.com/datasets/mmoreaux/environmental-sound-classification-50) и скачать файлы ваших данных.

---

Аудиоклип имеет одно временное измерение (с возможным вторым измерением для каналов, например, моно или стерео). В нашем случае мы будет работать над аудиофайлом одного канала.
С помощью библиотеки *matplotlib & librosa* мы можем визуализировать наш wav-samples, посмотреть на его информативность и выполнить *pre-proccesing*.

Рис 1. В обычном представлении звук представляет собой синусоидоальную волну, где по центру располагается атмосферное давление, затем точки сжатия (compression) и точки рефракции, представляющие собой амплитуду.

Давайте воспользуемся такими библиотеками как *matplotlib & librosa & display.waveshow* и отобразим наши аудио клипы (часть кода)

Наш 5 секундный клип по-умолчанию имеет частоту дискретизации в 22050 Гц, это означает, что он состоит из 5 * 22 050 = 110 250 последовательных чисел, представляющих изменения давления воздуха.
Это дает нам очень мало информации, т.к по ней мы можем только понять насколько громким или тихим является клип в любой момент времени, но не как насыщены определенные частоты в аудиосигнале.

Распространенное решение этой проблемы состоит в том, чтобы взять небольшие перекрывающиеся фрагменты сигнала и пропустить их через *Быстрое преобразование Фурье* (БПФ).
Преобразование Фурье разлагает функцию сигнала на составляющие частоты. Таким образом мы сможем отобразить амплитуду каждой частоты преобразовав их в децибелы

# Часть 2
**Преобразование**

Рис STFT. Из этого он у нас получится спектрограмма. Давайте напишем код, который сможешь отобразить аудиоклип в STFT.

Наша анатомия слухового аппарата воспринимает звук неидеально восприятие высоты звука зависит не только от частоты колебаний, но и от уровня громкости звука и его тембра.
Именно для анализа аудио, учитывающего особенности человеческого слуха, была введена психофизическая единица высоты звука – мел.

Рис 3. Мел система & Mel-спектрограмма в 3D

Мы можем поступить следующим образом: преобразовать наш аудиоклип в *mel-спектрограмму* или получить *mel кепстральные коэффициенты (MFCC)*, что позволит нам представить аудио в виде, наиболее близком к восприятию звука человеком.
Давайте напишем функцию, которая на вход будет принимать файл и с использованием параметров, которая предоставляет нам библиотека *librosa*, будет преобразововать их в *Mel & MFCC*.

Отдельно хочу сказать, что сейчас я буду использовать частоту дискретизации по-умолчанию. (22050)

Для данного набора данных, я думаю, что будет более целесообразней использовать *mel-спектрограммы* (с подборкой лучших параметров) и линейку *ResNet* моделей (18, 50).  
Объяснение тому, что *MCFF* лучше себя проявляет себя в задачах речеобразования, распознавания слов.
(Где выборку составляют акустические волны созданные системой органов: легкими, бронхами и трахеей, а затем преобразуется в голосовом тракте)

# Часть 3

Первым делом я хотел бы разделить датасет на train & valid часть по колонке fold. 
Все файлы до fold == 5 уходят в train часть, а остальное в valid.

Далее:
    1. Я хотел бы загрузить файл и посмотреть на его Sample rate и длину аудио.
    2. Написать две функции, первая из которых будет обрезать лишнее из графика и делать image, а вторая преобразовывать файл в melspectrogram.
    (Вторая функция будет выполнять еще условия среза или padding'a, if shape > 5, else shape < 5)
    3. Написать класс с функциями на вход которому будет поступать датасет с wav файлами, а он будет преобразовывать их в melspectrogram в соответствии с категориями.

# Часть 4

Применяем наш класс к train & valid частям
Подгружаем модель
Наш оптимизатор будет работать таким образом, что на каждую 10 эпоху он будет замедлять скорость обучения (learning rate).

---    

# Результат 

График train loss & valid loss

Лучший результат valid accuracy = 0.8025

Результат был достигнут с помощью модели ResNet18,(ResNet34 показала плохие результаты). 
Но не стоит забывать о параметрах, которые нам предлагает библиотека librosa при пре-процессинге аудиоклипа в melspectrogram'y.

Список наиболее удачных параметров, которые я использовал:


- sr=44100, 

- n_fft=16384,

- hop_length=308

- n_mels=256

- fmin=20

- fmax=17000

- top_db=80

- win_length=3969


P.s: К сожалению, ResNet50 запустить не удалось, даже с платной подпиской на GoogleColab. По многим найденным примерам ValidAccuracy мог достигать 0.87% и это превзошло бы модель которую использовал я.

Но вы можете попробовать запустить ее :)

--- 

В конце вы можете испытать модель на своих файлах, для этого будет достаточно скачать любой айдиоклип, который будет в зоне ваших классов.
