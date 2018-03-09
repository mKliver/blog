---
title: Публикация TAB на Mapserver
---
Документация Mapserver сообщает, что он может публиковать TAB файлы с [ОРИГИНАЛЬНОЙ СТИЛИЗАЦИЕЙ](http://demo.mapserver.org/cgi-bin/mapserv?map=%2Fosgeo%2Fmapserver%2Fogr-demos%2Fyk_demo%2Fyk.map&layer=lakes&layer=streams&zoomsize=2&zoomdir=1) из Mapinfo.
Что же посмотрим. 

Установка и начальная настройка
---------------------

Ставить буду на Windows, поэтому качаем [MS4W](https://www.ms4w.com/). Распаковываем и запускаем apache-install.bat,
В Службах должен появиться Apache.
Создаем папку для конфигов \ms4w\apps\example\, где и создаем файл расширения _map_.
Засовываем внутрь простой конфиг:


{% highlight xml linenos %}
MAP
  NAME           "WMS"
  STATUS         ON
  IMAGETYPE      PNG
  EXTENT         59.9 56.5 61.0 57.0
  SIZE           1000 1000
  SHAPEPATH      "C:/ms4w/apps/example/shp/"
  IMAGECOLOR     255 255 255
  CONFIG "MS_ERRORFILE" "D:\dev\Mapserver\ms4w\tmp\ms_error.txt"

	OUTPUTFORMAT
	   NAME "png"
	   DRIVER AGG/PNG
	   MIMETYPE "image/png"
	   IMAGEMODE RGB
	   EXTENSION "png"
	   FORMATOPTION "GAMMA=0.75"
	END
	
	WEB
		IMAGEPATH '/tmp/'

		IMAGEURL '/tmp/'

		METADATA
		  ows_title          'Simple map'
		  ows_onlineresource  
			'http://localhost:8081/cgi-bin/mapserv.exe?map=D:/dev/Mapserver/ms4w/apps/example/test.map&'
		  wms_getfeatureinfo         
			'http://192.168.70.18/cgi-bin/mapserv.exe?map=C:/ms4w/apps/example/test.map&'
		  wms_featureinfoformat      "text/plain"
		  wms_title          "Test WMS"
		  wms_srs            "EPSG:4326 EPSG:3857"
		END
		TEMPLATE 'fooOnlyForWMSGetFeatureInfo'
	END
  
	LAYER
	  NAME GREEN_POLYGON
	  TYPE POLYGON
	  TEMPLATE "dummy"
	  STATUS DEFAULT
	  CONNECTIONTYPE OGR
	  CONNECTION "d:\dev\tabs\GREEN_POLYGON.tab"
	  STYLEITEM "AUTO"
	  CLASS
	  END
	END
END
{% endhighlight %}

Так же как и в примерах, я добавляю тег _STYLEITEM "AUTO"_ к слою, в надежде увидить искомую оригинальную стилизацию.
Результат посмотреть можно поссылке:

{% highlight xml %}
http://localhost:8081/cgi-bin/mapserv.exe?map=D:/dev/Mapserver/ms4w/apps/example/test.map&mode=map&LAYERS=GREEN_POLYGON
{% endhighlight %}
 
Использование своей СК
---------------------

И увидел я белую картинку, ну и правильно, данные то у меня в мск. Добавляем новую СК в Mapserver.
Редактируем файл _\ms4w\proj\nad\epsg_ и добавлем в него строки:

{% highlight xml %}
# MSK
<100000> +proj=...+no_defs  <>
{% endhighlight %}

Все теперь можем показывать и ипользовать данные в нестандартной СК. Для этого в блоки _MAP_ и _LAYER_ добавялем:

{% highlight xml %}
PROJECTION
  "init=epsg:100000"
END
{% endhighlight %}

А еще добавим СК в _wms_srs_.

Теперь нужен EXTENT. Я поступил просто и открыл файлы в QGIS, прикинув на глаз интересующую территорию.
Попутно я узнал, что в табах не записана система координат. Поэтому запишу ее через _ogr2ogr_.

{% highlight xml %}
@echo off
set ext=.tab
FOR /f "tokens=*" %%G IN ('dir /b "D:\dev\tabs\*.tab"') 
	DO (ogr2ogr -a_srs "+proj=..." -f "MapInfo File" "D:\dev\tabs\projected\%%~nG%ext%" "D:\dev\tabs\%%~nG%ext%")
{% endhighlight %}

Стилизация
---------------------

После проделанных операций у меня наконец-то появилась картинка. Это был просто кусок зеленки с однородной заливкой.
Ничего сложного вроде, но нас больше интересует как он справиться с симболазерами Mapinfo. Поэтому покопавшись, я нашел железную дорогу.
Вот так это выглядит в Mapinfo:

![Road MapInfo](/blog/img/2018-03-09/road_mapinfo.jpg "Road MapInfo")

А вот так опубликовалось на Mapserver:

![Road Mapserver](/blog/img/2018-03-09/road_mapserver.jpg "Road Mapserver")

Как видно мапинфошные симболайзеры не перенеслись.
Что можно сделать? Стандартные символы MapInfo, которые имеют имена _mapinfo-sym-##_ описаны в файле в документации.
Скачаем его и подключим в секцию _MAP_:

{% highlight xml %}
SYMBOLSET      "D:\dev\Mapserver\ms4w\apps\etc\symbols_mapinfo.txt"
{% endhighlight %}

К сожалению мне это не помогло, потому что посмотреть стиль слоя через _ogrinfo_, я увидел следующее:

{% highlight xml %}
PEN(w:1px,c:#000000,id:"mapinfo-pen-73,ogr-pen-0")
{% endhighlight %}

Документации, как это описать я не нашел. Конечно можно взять и писать стили для каждого слоя, но идея-то не в этом.
Я то, хотел что бы стили сами подсосались из табов. Поэтому эта часть все еще в работе.

Человеческие ссылки
---------------------

Хоть меня и постигла неудача, но работу я продолжил. Далее я захотел подключить WMS на свой основной клиент написаный на _BruTile_.
И столкнулся с тем, что у меня все падает при запросе GetMap. Все дело в том, что стандартная url на wms уже содержит в себе параметры,
библиотека же к такому не готова.
Тут нам поможет Apache, на котором и крутится Mapserver. Пишем короткий конфиг:

{% highlight xml %}
Alias /wmsmap "/ms4w/Apache/cgi-bin/mapserv.exe"
<Location /wmsmap>
   SetHandler cgi-script
   Options ExecCGI
   SetEnv MS_MAPFILE "/ms4w/apps/example/test.map"
</Location>
{% endhighlight %}

И сохраняем в _\ms4w\httpd.d\_. Открываем _\ms4w\Apache\conf\httpd.conf_, находим строку 

{% highlight xml %}
# parse MS4W apache conf files
include "D:/dev/Mapserver/ms4w/httpd.d/httpd_*.conf"
{% endhighlight %}

И удаляем маску файла, так что бы подсасывались все conf файлы из этой папки. Основная идея подхода, в том что бы для каждого map файла
написать по конфигу и у все будет своя url без параметров.
Изначально хотел работать через теги <Directory>, как и указано в документации Mapserver, но там возникла непонятная проблема с 
правами на папку /tmp/.

После этого урла до wms стала http://localhost:8081/wmsmap, что уже по человечески. 

Заключение
---------------------

Ну что сказать, толи я дурак, толи лыжи не едут. Но получить, что я хотел не получилось. Мечты о картинке выглядещей так же как в Mapinfo не осуществились.
На этом я продолжил публиковать табы, и пока не оставляю надежды найти способ описания симболайзеров из Mapinfo.


