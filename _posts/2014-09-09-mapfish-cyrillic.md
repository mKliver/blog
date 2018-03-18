---
title: Кириллица в Mapfish print plugin
---

Сразу скажу, что способ приведенный ниже работает только в случае если надписи содержащие кириллицу приходят с клиента в виде параметров.
 Те надписи, которые захардкожены в конфиге печати все равно отображаются краказябрами.
 Для отображения кириллицы я использую кодировку __Identity-H__. Но ее одной недостаточно, если попробовать напечатать такой пример:


{% highlight sql linenos %}
   - !text
       text: "Привет"
       fontEncoding: Identity-H
       align: center
{% endhighlight %}

То ничего не напечатается, будет просто пустое место.
Для решения этой проблемы нужно подключить шрифт работающий с данной кодировкой. Мой выбор пал на [FreeSans](http://www.fontspace.com/gnu-freefont/freesans). Подключаем шрифт в отдельном блоке:

{% highlight sql linenos %}
scales:
   ....
fonts: 
    - 'E:\GeoServer 2.5.1\data_dir\printing\FreeSans.ttf'
    - 'E:\GeoServer 2.5.1\data_dir\printing\FreeSansOblique.ttf'
hosts:
   ....
layouts:
   ....
{% endhighlight %}

Подключение шрифта к элементу:

{% highlight xml %}
   - !text
       text: "Привет"
       fontEncoding: Identity-H
       font: FreeSans
       align: center
{% endhighlight %}

Теперь все должно работать.<br>
Так же скажу, что подключить шрифты к легенде мне не удалось. Легенда выводится криво - имена стилей показывают кириллицу, а вот имена слоев нет.</br>
В работе использовалась связка Geoserver 2.5.2 и Print plugin 1.2.

 
