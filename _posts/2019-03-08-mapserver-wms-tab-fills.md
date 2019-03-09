---
title: Публикация TAB на Mapserver
tags: [mapserver, mapinfo]
---
Ранее я повторил опыт из документации Mapserver, и опубликовал tab файлы в надежде, что Mapserver их сам автоматически стилизует.
Этого, конечно, не произошло и результат был не очень. Mapserver справился с простой стилизацией, т.е. обычными заливками, простыми линиями. 
Но все остальное либо не стилизовалось, либо стилизовалось в нечно невнятное. Но все можно исправить(нет). Посмотрим, что можно сделать с заливками.

Установка и начальная настройка
---------------------

Подключим слой в конфиг


{% highlight xml linenos %}
 LAYER
  NAME GIDROGRAFI_POLYGON
  TYPE POLYGON
  TEMPLATE "dummy"
  STATUS DEFAULT
  CONNECTIONTYPE OGR
  CONNECTION "c:\MapServer\Belokamenii500\projected\GIDROGRAFI.TAB"
  DATA "SELECT * FROM GIDROGRAFI WHERE OGR_GEOMETRY='POLYGON' OR OGR_GEOMETRY='MULTIPOLYGON'"
  STYLEITEM "AUTO"
  CLASS
  END
  PROJECTION
    "init=epsg:100000"
  END
END # Layer

{% endhighlight %}

Так как в map файле намешаны различные типы геометрий, то я отфильтровал только полигоны. В дальнейшем можно подключить этот же файл в конфиг, оставив другой тип геометрии и стилизовать его по-своему.
И увидел я ничего. Я знал, что у меня в слое есть болота нарисованыне штриховой заливкой, с ней-то и не справился Mapserver. Ну чтож поможем ему.

Сначала нужно узнать имя заливки. Для этого воспользуемся _ogrinfo_:

{% highlight xml linenos %}
ogrinfo C:\tabs\RASTITELJNOSTJ_I_GRUNTI.TAB RASTITELJNOSTJ_I_GRUNTI -sql "SELECT * FROM RASTITELJNOSTJ_I_GRUNTI WHERE OGR_GEOMETRY='POLYGON'" > C:\tabs\out.txt
{% endhighlight %}

В выводе получаем что-нибудь такое:

{% highlight xml %}
OGRFeature(GIDROGRAFI):196
  id (Integer) = 196
  Style = BRUSH(fc:#000000,id:"mapinfo-brush-3,ogr-brush-2");PEN(w:0pt,c:#000000,id:"mapinfo-pen-1,ogr-pen-1")
  POLYGON ((...))

OGRFeature(GIDROGRAFI):197
  id (Integer) = 197
  Style = BRUSH(fc:#000000,id:"mapinfo-brush-3,ogr-brush-2");PEN(w:0pt,c:#000000,id:"mapinfo-pen-1,ogr-pen-1")
  POLYGON ((...))

{% endhighlight %}

В данном случае нас интересует мапинфошная заливка с именем _mapinfo-brush-3_.
Далее открываем tab файл в Mapinfo и ищем объект по _id_ смотрим как он нарисован и рисуем в графическом редакторе такую же картинку размером 32*32 и сохраняем в png.

![Hatch example](/blog/img/2019-03-08/map-brush-3.png "Hatch example")

Теперь научим Mapserver понимать как использовать эту картинку.
В прошлый раз в описании конфига мы подключали файл ms4w\apps\etc\mapinfo_symbols.txt, который по-идее хранит описание стилей, которые использует автостилизатор. Добавим в файл следующее:

{% highlight xml %}
SYMBOL
  NAME "mapinfo-brush-3"
  TYPE PIXMAP
  IMAGE "map-brush-3.png"
  TRANSPARENT 0
END
{% endhighlight %}

Тут всего-то и указывается путь к файлу заливки(у меня они все лежат в одной директории с файлом стилей) и указывается какому стилю этот файл заливки соответствует.

На это собственно настройка закончена. И открыв карту я таки увидел свои болота.

![Result stylization](/blog/img/2019-03-08/fill_result.jpg "Result stylization")

Заключение
---------------------

Стилизация заликов оказалась условно автоматической, но она работает. Конечно приходится проводить изрядное количество подготовительных работ, вроде рисования картинок и поиска имен этих заликов.