# Sipp

В этом репозитории есть реализация алгоритма планирования траектории в среде с динамическими препятствиями Sipp и его модификации Anytime Sipp, визуализатор работы алгоритма. Реализация протестирована на большой выборке тестов, сгенерированных рандомно. Результаты тестирования также доступны в репозитории в [ноутбуке](Tests/analysis.ipynb).
Примеры заданий, результатов работы алгоритмов и видео можно найти [здесь](Examples/).

## Содержание

* [Examples](#examples)
* [Постановка задачи](#постановка-задачи)
* [Алгоритм Sipp и Anytime Sipp](#алгоритм-sipp)
* [Чекер](#чекер)
* [Визулизация](#визулизация)
* [Тестирование и его результаты](#тестирование-и-его-результаты)

## Examples
Директория содержит 4 папки:
- tasks - задания. Подробнее о том, в каком виде должно быть представлено задание, написано [ниже](#вид-входных-данных).
- logs - решения этих заданий алгоритмом Sipp
- logs_anytime - решения этих заданий алгоритмом Anytime Sipp
- videos - визуализация найденого пути алгоритмом Sipp

Можно прогнать эти тесты с помощью команды (Запустятся алгоритмы на заданиях, и чекер на полученных результатах):
```bash
ctest -R 'examples'
```
Вывод будет такой:
```
    Start 1: examples
1/2 Test #1: examples .........................   Passed    0.65 sec
    Start 2: examples/anytime
2/2 Test #2: examples/anytime .................   Passed    0.88 sec

100% tests passed, 0 tests failed out of 2

Total Test time (real) =   1.53 sec
```

## Постановка задачи
Пусть есть некоторая среда  – двумерное поле со свободными и занятыми клетками, где занятые клетки – статические препятствия. На этом поле также есть динамические препятствия. У них нет размеров, поэтому будет считать, что это точки. Динамические препятствия двигаются равномерно и прямолинейно между клетками с общей стороной, причем из любой клетки они могут переходить только в 4 направлениях: вверх, вниз, вправо, влево. Переход между клетками занимает 1 единицу времени. При целом числе единиц времени любое динамическое препятствие должно находиться в центре какой-нибудь клетки.

![directions](/Images/directions.png)

Есть агент, для которого нужно найти путь из данной клетки S (start) в данную клетку F (finish). У агента нет размеров, поэтому тоже считаем, что агент – точка. Агент двигается равномерно и прямолинейно только по свободным клеткам, и так же, как и динамические препятствия, может переходить между клетками с общей стороной только в тех же 4 направлениях. Аналогично динамическим препятствиям, агент перемещается в соседнюю клетку за 1 единицу времени, и при целом числе времени должен находиться в центре какой-нибудь клетки.

**Задача**: для агента нужно найти путь из клетки S в клетку F, проходящий только по свободным клеткам, так что в любой момент времени (необязательно целый) координаты агента (необязательно целые) не совпадают ни с каким динамическим препятствием.
![grid](/Images/grid.png)

На рисунке пример поля на старте, белые клетки – свободные, черные – занятые. Красная точка – искомый агент, стоит на старте. Клетка, помеченная буквой F, – финиш, куда нужно прийти агенту. 3 синие точки – динамические препятствия на своих стартовых позициях.

## Алгоритм Sipp и Anytime Sipp
### Пререквизиты
Необходимо установить библиотеку [Boost](https://www.boost.org)
### Сборка:
```bash
mkdir Build
cd build
cmake ..
make SippSearch
```

### Запуск:
Находимся в директории Build.
```bash
./Src/Planning/SippSearch input.xml [-w hweight] [-l logLevel]
```
В аргументах командной строки нужно передать путь от Build до xml-файла с заданием либо до директории с заданиями (тогда сразу протестируются все задания из этой директории).
Если передать параметр hweight - начальный вес эвристики - запустится алгоритм Anytime Sipp. Если этого параметра не будет,то Sipp.
Опционально, можно передать уровень логирования (logLevel):
- -1 - в консоль ничего не выведится
- 0 - только путь до файла с заданием или пути, если был передан путь до директории
- 1 - выведутся результаты (основные показатели: pathlength, nodescreated, numberofsteps, searchtime)
- 2 - всё, что в предыдущих, а также слова, сигнализирующие о конце какого-то этапа работы программы
- 3 - всё, что в предыдущих, а также на каждом шаге алгоритма будут выводиться параметры текущего node и списки безопасных интервалов его successors
#### Пример
```bash
./Src/Planning/SippSearch ../Examples/tasks/        # запуск Sipp на всех тестах, лежащих в директории ../Examples/tasks/
./Src/Planning/SippSearch ../Examples/tasks/ -w 2   # запуск Anytime Sipp на тех же тестах
```
### Вид входных данных
- \<map\> – тег открывает секцию, описывающую карту
- \<height\> и \<width\>  – высота и ширина карты соответственно.
- \<startx\>, \<starty\> – координаты старта агента, для которого будет строиться путь
- \<finishx\>, \<finishy\> – координаты финиша агента
- \<grid\> – описание свободных и занятых статическими препятствиями клеток карты.
- \<row\>  – одна строчка карты, значение клетки может быть равным 1, если в клетке статическое препятствие, или 0, если клетка свободна
- \<dynamicobstacles\> - описание траекторий движения динамических препятствий
- \<obstacle\> – внутри этого тега описание пути препятствия, таких тегов может быть несколько в теге \<dynamicobstacles\>

Траектория пути представляется в виде последовательности троек из целых неотрицательных чисел (x, y, time) – координаты клетки и время, в которое препятствие или агент там находится. Эта тройка записывается в XML файле как атрибуты тега \<point\>. Любые 2 соседние тройки должны образовывать либо прямолинейный отрезок пути так, что время движения между этими координатами должно быть в точности равно разнице координат времени в этих тройках, либо координаты клетки в соседних тройках должны быть равны, а координаты времени отличаются на целое число больше 0.

#### Пример представления пути
![path_representation](/Images/path_representation.png)

Пусть на рисунке траектория движения препятствия (или агента), начинающаяся в клетке (3, 2), конец – в клетке (0, 0). Пусть также в клетке (2, 1), он ждал 2 единицы времени. Тогда будет записано так:
```xml
<obstacle id="1">
    <point x="3" y="2" time="0"/>
    <point x="3" y="1" time="1"/>
    <point x="2" y="1" time="3"/>
    <point x="2" y="1" time="5"/>
    <point x="0" y="1" time="7"/>
    <point x="0" y="0" time="8"/>
 </obstacle>
```

## Генерация тестов
### Сборка и запуск
Находимся в директории Build

Сборка:
```bash
make Generate
```
Запуск:
```bash
./Tests/Generate maps_directory tasks_directory a_1 a_2 a_3 ... a_n
```
Для каждой карты из директории maps_directory генерирует по 5 тестов с a_i количеством динамических препятствий, все тесты кладет в директорию tasks_directory. Здесь карта - это файл, строго соблюдающий следующий формат:
```txt
type octile
height 4
width 5
map
...11
...11
11.11
11...
```
После слов height и width положительные числа, равные высоте и ширине карты соответственно. После строки "map" идет описание карты: количество строк равно высоте карты, количество символов в каждой строке ровно ширине карты. Символ '.' означает пустую клетку, любой другой непробельный символ - занятая клетка.

#### Пример
Тесты на городских картах размера 256x256 с маленьким числом динамических препятствий(10, 20, 50, 100 и 200), лежащие в [папке](TestsData/City/tests_256/small_tasks/tasks), были сгенерированы следующей командой:
```bash
./Tests/Generate ../TestsData/City/tests_256/maps/ ../TestsData/City/tests_256/small_tasks/tasks/ 10 20 50 100 200
```
## Чекер
### Сборка и запуск
```bash
make Check
```
```bash
./Tests/Check logs_directory
```
logs_directory - директория с файлами, полученными на выходе при работе программы SippSearch. Проверяется, что все полученные пути агентов записаны в правильном виде, действительно начинаются и заканчиваются в клетках старта и финиша соответственно, а также не пересекают ни статические, ни динамические препятствия.

## Визулизация
```bash
Python3 ../Src/Visualization/visualize.py log_file.xml [-o output_file.mp4]
```
log_file.xml - файл, полученный работой Sipp. Опционально, можно передать путь, куда будет записнао видео. По умолчанию, оно появляется в той же директории, что и log_file.xml

Примеры визуализаций можно найти [здесь](/Examples/videos/)
На всех видео синими точками обозначены динамические препятствия, красной - агент, для которого ищется путь. Белые клетки - свободные, черные - занятые.

## Тестирование и его результаты
Были сгенерированы тесты на картах размеров 256x256, 512x512 и 1024x1024 (по 30 карт каждого размера), для каждой из карт было сгенерировано задание с 10, 20, 50, 100, 200, 500, 1000 динамических препятствий (по 5 тестов для каждой конфигурации). Итого получилось 3150 тестов.
Результаты тестирования оформлены в качестве таблиц и графиков в [ноутбуке](Tests/analysis.ipynb).

Запустить алгоритм поиска пути sipp и чекер, проверяющий полученный результат на всех 3150 заданиях, лежащих в репозитории, можно следующей командой:
```bash
ctest -R 'run'
```
Результат работы будет примерно таким:
```bash
Test project
      Start  1: preparation/run
 1/10 Test  #1: preparation/run ..................   Passed    0.01 sec
      Start  2: 256/small_tasks/run
 2/10 Test  #2: 256/small_tasks/run ..............   Passed  150.37 sec
      Start  3: 256/tasks_500/run
 3/10 Test  #3: 256/tasks_500/run ................   Passed  170.99 sec
      Start  4: 256/tasks_1000/run
 4/10 Test  #4: 256/tasks_1000/run ...............   Passed  385.77 sec
      Start  5: 512/small_tasks/run
 5/10 Test  #5: 512/small_tasks/run ..............   Passed  426.71 sec
      Start  6: 512/tasks_500/run
 6/10 Test  #6: 512/tasks_500/run ................   Passed  384.81 sec
      Start  7: 512/tasks_1000/run
 7/10 Test  #7: 512/tasks_1000/run ...............   Passed  779.00 sec
      Start  8: 1024/small_tasks/run
 8/10 Test  #8: 1024/small_tasks/run .............   Passed  1696.57 sec
      Start  9: 1024/tasks_500/run
 9/10 Test  #9: 1024/tasks_500/run ...............   Passed  1451.54 sec
      Start 10: 1024/tasks_1000/run
10/10 Test #10: 1024/tasks_1000/run ..............   Passed  3027.33 sec

100% tests passed, 0 tests failed out of 10

Total Test time (real) = 8473.15 sec
```
Команды, выполняющиеся в каждом тесте, можно посмотреть [здесь](/Tests/CMakeLists.txt).
Обратите внимание на время тестирования! Из-за большого числа тестов (3150) оно довольно большое. Если не хочется ждать, можно запустить меньшее число тестов, заменив 'run' на часть имени или полное имя теста. Например, прогоним один блок тестов, где проверим работу на картах 256x256 c маленьким числом динамических препятствий (до 200), то есть программа будет запущена всего на 750 заданиях, а время работы этого блока тестов не более 5 минут:
```bash
ctest -R '256/small_tasks/run'
```

Либо, если не хочется ждать, можно запустить блоки тестов, в которых не планируются траектории, а только запускается чекер, проверяющий уже полученные мной результаты работы алгоритма, лежащие в репозитории. Здесь проверяется, что все полученные пути агентов записаны в правильном виде, действительно начинаются и заканчиваются в клетках старта и финиша соответственно, а также не пересекают ни статические, ни динамические препятствия.

```bash
ctest -R 'check'
```

В консоли видим примерно такой результат:
```bash
Test project
    Start 11: 256/small_tasks/check
1/9 Test #11: 256/small_tasks/check ............   Passed   20.78 sec
    Start 12: 256/tasks_500/check
2/9 Test #12: 256/tasks_500/check ..............   Passed   17.12 sec
    Start 13: 256/tasks_1000/check
3/9 Test #13: 256/tasks_1000/check .............   Passed   39.78 sec
    Start 14: 512/small_tasks/check
4/9 Test #14: 512/small_tasks/check ............   Passed   49.89 sec
    Start 15: 512/tasks_500/check
5/9 Test #15: 512/tasks_500/check ..............   Passed   27.52 sec
    Start 16: 512/tasks_1000/check
6/9 Test #16: 512/tasks_1000/check .............   Passed   53.43 sec
    Start 17: 1024/small_tasks/check
7/9 Test #17: 1024/small_tasks/check ...........   Passed  163.67 sec
    Start 18: 1024/tasks_500/check
8/9 Test #18: 1024/tasks_500/check .............   Passed   59.60 sec
    Start 19: 1024/tasks_1000/check
9/9 Test #19: 1024/tasks_1000/check ............   Passed   93.41 sec

100% tests passed, 0 tests failed out of 9

Total Test time (real) = 525.22 sec
```