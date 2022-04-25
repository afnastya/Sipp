# Sipp
В этом репозитории есть реализация алгоритма планирования траектории в среде с динамическими препятствиями Sipp, визуализатор работы алгоритма. Реализация протестирована на большой выборке тестов, сгенерированных рандомно. Результаты тестирования также доступны в репозитории.

## Алгоритм Sipp
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
./Src/Planning/SippSearch input.xml [logLevel]
```
В аргументах командной строки нужно передать путь от Build до xml-файла с заданием либо до директории с заданиями (тогда сразу протестируются все задания из этой директории). Опционально, последним аргументом можно передать уровень логирования:
- -1 - в консоль ничего не выведится
- 0 - только путь до файла с заданием или пути, если был передан путь до директории
- 1 - выведутся результаты (основные показатели: pathlength, nodescreated, numberofsteps, searchtime)
- 2 - всё, что в предыдущих, а также слова, сигнализирующие о конце какого-то этапа работы программы
- 3 - всё, что в предыдущих, а также на каждом шаге алгоритма будут выводиться параметры текущего node и списки безопасных интервалов его successors
## Генерация тестов
Находимся в директории Build
Сборка:
```bash
make Generate
```
Запуск:
```bash
./Tests/Generate maps_directory tasks_directory a_1 a_2 a_3 ... a_n
```
Для каждой карты из директории maps_directory генерирует по 5 тестов с a_i количеством динамических препятствий, все тесты кладет в директорию tasks_directory
## Чекер
```bash
make Check
./Tests/Check logs_directory
```
## Визулизация
```bash
Python3 ../Src/Visualization/visualize.py log_file.xml
```
log_file.xml - файл, полученный работой Sipp

## Тестирование и его результаты
Были сгенерированы тесты на картах размеров 256x256, 512x512 и 1024x1024 (по 30 карт каждого размера), для каждой из карт было сгенерировано задание с 10, 20, 50, 100, 200, 500, 1000 динамических препятствий (по 5 тестов для каждой конфигурации). Итого получилось 3150 тестов. Результаты тестирования оформлены в качестве таблиц и графиков в ноутбуке Tests/analysis.ipynb.