# SWaT anomaly detecion

Исследование для магистерской диссертации на тему обнаружения аномалий в технологических процессах с использованием LSTM нейронных сетей.

## Описание набора данных

В работе используется [набор данных SWaT](https://itrust.sutd.edu.sg/itrust-labs-home/itrust-labs_swat/) версии SWaT.A1_Dec 2015. Набор данных представляет из себя собранные в течение 11 суток записи сигналов с датчиков и исполнительных механизмов в системе очистки воды. Для получения набора данных исследователи создали физическую лабораторую установку по очистке воды ([Видео на YouTube](https://www.youtube.com/watch?v=i4vCG4clNZQ)).

![SWaT testbed](https://itrust.sutd.edu.sg/wp-content/uploads/2015/03/testbed-1.png)

Технологический процесс очистки воды состоял из 6 стадий, на каждой из которых работали различные датчики (уровня воды, давления, химического состава) и исполнительные механизмы (насосы, заслонки, перемешивающие устройства). На каждой стадии техпроцесса датчиками и актуаторами управлял ПЛК. Все ПЛК были объединены в сеть и подключены к SCADA-системе. Данные о состоянии 51 датчика и актуатора ежесекундно фиксировались и записывались в базу данных.

При создании набора данных система 7 суток работала в режиме нормального функционирования и 4 суток под воздействием моделируемых кибератак. Моделируемые атаки иммитировали ситуацию, когда злоумышленник получил доступ к ручному изменению показаний датчиков и изменению состояний исполнительных механизмов. Например, производилось умышленное занижение показаний датчика уровня воды в баке для того, чтобы насос не прекращал подачу воды, что могло привести к переполнению бака и нарушению техпроцесса.

Процесс сбора данных и смоделированные атаки описаны в [статье](https://www.researchgate.net/publication/305809559_A_Dataset_to_Support_Research_in_the_Design_of_Secure_Water_Treatment_Systems).

## Данные для исследования

Целью исследования была разработка метода обнаружения аномалий в технологическом процессе с помощью рекуррентной нейронной сети с LSTM. 
Для целей исследования были выбраны сигналы трех датчиков:
* LIT101 - датчик уровня воды на первой стадии техпроцесса
* DPIT301 - датчик перепада давления на третьей стадии техпроцесса
* LIT301 - датчик уровня воды на третьей стадии техпроцесса

В качестве обучающих данных были выбраны данные за период 3 суток с 10:00:00 25.12.2015 по 09:59:59 28.12.2015 (Обучающие данные находятся в [__Train.csv__](/Train.csv)).

В качестве тестовых данных с аномалиями были выбраны данные за период 2 суток с 10:00:00 28.12.2015 по 09:59:59 30.12.2015 (Обучающие данные находятся в [__Test.csv__](/Test.csv)).

За выбранный период на тестовые сигналы было проведено 6 атак: по 2 на каждый сигнал.

![Train and Test](/pics/Train_and_Test_3_signals.png)


|     Точка атаки    |     Действие                            |     Дата          |     Время начала    |     Время окончания    |
|--------------------|-----------------------------------------|-------------------|---------------------|------------------------|
|     LIT101         |     Увеличение на 1 каждую   секунду    |     28.12.2015    |     11:20:09        |     11:28:02           |
|     LIT101         |     Установка значения 700              |     29.12.2015    |     18:29:59        |     18:41:40           |
|     DPIT301        |     Установка значения 45               |     28.12.2015    |     13:09:45        |     13:25:54           |
|     DPIT301        |     Установка значения 45               |     30.12.2015    |     01:42:07        |     01:53:30           |
|     LIT301         |     Установка значения 1200             |     28.12.2015    |     12:08:05        |     12:15:12           |
|     LIT301         |     Уменьшение на 1 каждую   секунду    |     29.12.2015    |     11:57:04        |     12:01:44           |


## Метод обнаружения аномалий 

Для обнаружения аномалий нейронная сеть обучалась на данных из режима нормального функционирования в режиме автокодировщика. То есть на вход модели последовательно подавались отсчеты сигнала, и она обучалась воспроизводить этот же отсчет на выходе. Идея состояла в том, что рекуррентная нейросеть с LSTM за счет памяти, присущей таким сетям, сможет выучить паттерн, прослеживающися в сигнале нормы и успешно его воспроизводить. В случае нарушения выученного паттерна, сеть бы воспросизводила отсчеты с большей ошибкой, и по сигналу ошибки воспроизведения было бы возможным обнаружение аномалий.

После обучения на вход сети подавался весь обучающий сигнал и вычислялся сигнал ошибки воспроизведения (модуль разности между входом и выходом). Максимальное значение сигнала ошибки воспроизведения на обучающем сигнале принималось в качестве порога обнаружения аномалий.

Для тестирования на нейронную сеть подавался тестовый сигнал, рассчитывался сигнал ошибки воспроизведения, и те отсчеты, где сигнал ошибки воспроизведения превышал установленный порог, считались аномальными. Так как для каждого отсчета было известно, является ли он в действительности аномальным или нет, было возможным рассчитать показатели качества обнаружения аномалий на тестовой выборке (TP, TN, FP, FN, accuracy, precision, recall).

Метод обнаружения аномалий более подробно описан в [статье](https://elibrary.ru/download/elibrary_48086972_37552010.pdf).

Для повышения точности обнаружения аномалий дополнительно сигнал ошибки воспроизведения на тестовой выборке обрабатывался с помощью [алгоритма обнаружения голосовой активности](https://moluch.ru/archive/28/3172/) (Voice Activity Detection, VAD). Идея применения этого алгоритма состояла в том, что сигнал ошибки на учасках нормы подобен участкам тишины (фонового шума) в голосовом сигнале, а на участках аномалий активность сигнала можно обнаружить подобно участкам речевой активности. После применения этого алгоритма также рассчитывались показатели качества обнаружения аномалий.


## Проведенные эксперименты

В рамках исследования было проведено 5 экспериментов. Сначала обучалось по одной нейросетевой модели для каждого из трех сигналов и проводилось тестирование качества обнаружения аномалий. Данные эксперименты представлены в блокнотах

1. [__LIT101.ipynb__](/LIT101.ipynb)
2. [__DPIT301.ipynb__](/DPIT301.ipynb)
3. [__LIT301.ipynb__](/LIT301.ipynb)

После этого был проведен эксперимент с обучением одной нейросетевой модели сразу для трех сигналов (на вход подавался многомерный временной ряд). При этом на выходе модели анализировались как сигналы ошибки воспроизведения для каждого канала по-отдельности, так и суммарный сигнал ошибки. Данный эксперимент представлен в блокноте 

4. [__Multisignal.ipynb__](/Multisignal.ipynb)

Дополнительным экспериментом было обучение модели на сигнале LIT301, в котором было смоделировано наличие в техпроцессе двух режимов, каждый из которых считается нормальным. Для моделирования второго режима в обучающем сигнале определенный временной участок был намеренно поднят на 1000 единиц. В тестовый сигнал также был внесен такой же второй режим, а также две аномалии: по одной в каждом режиме. При первой аномалии уровень повышается на 500 мм, а при второй – снижается на 400 мм

![Train Multimode](/pics/Miltimode_train.png)
![Test Multimode](/pics/Miltimode_test.png)

Данные для этого эксперимента представлены в [__Train multimode.csv__](/Train%20multimode.csv), [__Test multimode.csv__](/Test%20multimode.csv)

Данный эксперимент представлен в блокноте 

5. [__Multimode LIT301.ipynb__](/Multimode%20LIT301.ipynb)

## Результаты

Результаты тестов с выводом показателей качества обнаружения аномалий приведены в конце каждого блокнота с экспериментами.

## Примечания

Эксперименты реализованы программно в среде Jupyter Notebook на Python 3 с применением библиотек Pandas, Numpy, Scikit-learn, Matplotlib, Tensorflow, Keras.

Дополнительно разработаны модули на Python:
* [__rnn_training.py__](/rnn_training.py) - создание и обучение модели рекуррентной нейронной сети
* [__VAD.py__](/VAD.py) - реализация алгоритма Voice Activity Detection
* [__cf_matrix.py__](/cf_matrix.py) - отрисовка матрицы ошибок (Confusion matrix) в виде Heatmap
