# duma2016
Скрипт для построения схемы одномандатных округов на выборах в Госдуму в 2016 году

Данные для построения
[Схема одномандатных округов для проведения выборов в Госдуму](http://gis-lab.info/qa/duma2016.html)

## Необходимые инструменты и данные

* perl, make, wget, ogr2ogr

* Для обрезки по береговой линии дополнительно нужна утилита osmfilter и shape-файл с береговой линией с сайта http://openstreetmapdata.com/data/land-polygons

## Использование

### Минимальная инструкция
* Поместить файлы из репозитория в отдельную директорию
* Распаковать туда shape-файлы `land_polygons.*` с береговыми границами с сайта http://openstreetmapdata.com/data/land-polygons
* Выполнить команды
```
cat id.txt | ./id2Makefile.pl > Makefile
cat Makefile.fix >> Makefile
make fix_okrug_23
make fix_okrug_69
make fix_okrug_70
make fix_okrug_222
make fix_okrug_223
make okrug_all
make okrug_all_land
```

На многоядерных и/или многопроцессорных системах время сборки можно сильно уменьшить, если использовать многопоточную сборку `make -j N`, где `N` - число потоков. Построение береговой линии России и последующая обрезка может занять много времени (час или дольше, в зависимости от мощности компьютера).

Результат будет содержаться в файлах `okrug_all_diss.*` `okrug_all_diss_land.*`

При построении границ всех избирательных округов скрипт скачивает с сервера openstreetmap.org чуть более 500Мб данных. Возможна ситуация, когда скачивание будет прервано сервером из-за превышения лимита. В этом случае необходимо удалить неудачно скачанный osm файл и перезапустить команду make.

### Создание Makefile
Сборка осущетсвляется с помощью команды `make`. Файл `Makefile` создается командой
```
cat id.txt | ./id2Makefile.pl > Makefile
```

Для правильного построения границы округов 69, 70, 222, 223 в `Makefile` нужно добавить содержимое файла `Makefile.fix`
```
cat Makefile.fix >> Makefile
```

### Описание возможных целей `make`

#### Работа с отдельными округами

* Создать shape-файл округа `NNN`
```
make okrug_NNN.shp
```

* Создать shape-файл округа `NNN` одним полигоном
```
make okrug_NNN_diss.shp
```

* Удалить все файлы, созданные при построении shape-файла округа `NNN` (скачанные `osm` файлы не удаляются)
```
make clean_NNN
```

* Удалить все файлы, созданные при построении shape-файла округа `NNN` вместе  со скачанными `osm` файлами
```
make clean_all_NNN
```

#### Исправление границ
В некоторых округах требуется ручное вмешательство для правильного проведения границ. 

* Собрать shape-файл с правильной границей округов 69, 70, 222, 223
```
make fix_okrug_23
make fix_okrug_69
make fix_okrug_70
make fix_okrug_222
make fix_okrug_223
```
После выполнения этих команд необходимо снова пересобрать файлы `okrug_23_diss.shp`, `okrug_69_diss.shp`, `okrug_70_diss.shp`, `okrug_222_diss.shp`, `okrug_223_diss.shp`. 

* Удаление shape-файлов, созданных для построения исправленной границы округов 23, 69, 70, 222, 223 (скачанные `osm` файлы не удаляются)
```
make clean_mordovia
make clean_surgut
make clean_khabarovsk
```

* Удаление shape-файлов, созданных для построения исправленной границы округов 23, 69, 70, 222, 223 вместе со скачанными `osm` файлами
```
make clean_surgut_all
make clean_khabarovsk_all
```

#### Работа со всеми округами

* Создать shape-файл с границами округов
```
make okrug_all
```

* Удалить все файлы, созданные при построении shape-файлов всех округов (скачанные `osm` файлы не удаляются и не удаляются shape-файлы с границей России)
```
make clean_okrug
```

* Удалить все файлы, созданные при построении shape-файлов всех округов вместе со скачанными `osm` файлами (shape-файлы с границей России не удаляются)
```
make clean_all_okrug
```

* Удалить все файлы, созданные при построении shape-файлов всех округов и все файлы, созданные при построении shape-файла береговой границы России (скачанные `osm` файлы не удаляются)
```
make clean
```

* Удалить все файлы, созданные при построении shape-файлов всех округов и все файлы, созданные при построении shape-файла береговой границы России вместе со скачанными `osm` файлами
```
make clean_all
```

* Скачать все osm файлы (за исключением файлов, которые скачиваются для исправления границ в округах 222 и 223)
```
make osm_source
```

#### Обрезка по береговой линии
* Собрать shape-файл с береговой границей России (требуется предварительно загрузить Land polygons с сайта http://openstreetmapdata.com/data/land-polygons)
```
make russia_land_diss.shp
```

* Обрезка округа по береговой линии
```
make okrug_NNN_diss_land.shp
```

* Обрезка всех округов по береговой линии
```
make make okrug_all_land
```

* Удалить все файлы, созданные при построении shape-файла береговой границы России (скачанные `osm` файлы не удаляются)
```
make clean_russia
```

* Удалить все файлы, созданные при построении shape-файла береговой границы России вместе  со скачанными `osm` файлами
```
make clean_all_russia
```

## Проверка
Полученный результат можно проверить на правильность несколькими способами

### Отсутствие дыр между округами
Проверить это можно следующим образом:

Объединить все округа в один полигон
```
ogr2ogr okrug_all_joined.shp okrug_all_diss.shp -dialect sqlite -sql "SELECT ST_Union(geometry) AS geometry FROM okrug_all_diss"
```
В полученном файле не должно быть дыр в странных местах (как это автоматизировать?).

### Отстутствие наложений округов

QGIS->Vector->Topology Checker. Выбрать слой okrug_all_diss.shp и правило "must not overlap" для проверки.

### Отстутствие ошибок геометрии
QGIS->Vector->Geometry Tools->Check Geometry Validity

## Упрощение геометрии

```
mapshaper okrug_all_diss_land.shp -simplify 30% keep-shapes -o okrug_all_diss_land-30.shp
mapshaper okrug_all_diss_land.shp -simplify 10% keep-shapes -o okrug_all_diss_land-10.shp
mapshaper okrug_all_diss_land.shp -simplify 1% keep-shapes -o okrug_all_diss_land-1.shp
mapshaper okrug_all_diss_land.shp -simplify 0.1% keep-shapes -o okrug_all_diss_land-01.shp

```
