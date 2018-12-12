---
title: Mapserver+nginx+FastCGI on Ubuntu
tags: [mapserver, nginx, linux]
---

I'm new with Linux. But try to tell how to start with Mapserver on Ubuntu. 

Installing
---------------------

At first add _UbuntuGIS_ repository

{% highlight xml linenos %}
sudo add-apt-repository ppa:ubuntugis/ubuntugis-unstable
sudo apt-get update
{% endhighlight %}

Before install MapServer don't forget that one needed in some libraries: 
curl, gdal,org, agg. More information about this on [Mapserver site]( http://mapserver.org/installation/unix.html#introduction)


{% highlight xml %}
sudo apt-get install libgd2-xpm-dev
sudo apt-get install cgi-mapserver mapserver-bin
sudo apt-get install nginx
sudo apt-get install spawn-fcgi
{% endhighlight %}
 
FastCGI config
---------------------

Create file /etc/init.d/mapserv/ and put into:

{% highlight xml %}
#! /bin/sh
#
# description: Mapserver Service Manager
# processname: lt-mapserv
# pidfile: /var/run/mapserv.pid
# Source function library.
#. /etc/init.d/functions
# Check that networking is up.
#. /etc/sysconfig/network
if [ "$NETWORKING" = "no" ]
then
        exit 0
fi
PREFIX=/usr
NAME=mapserv
PID=/var/run/mapserv.pid
DAEMON=$PREFIX/bin/spawn-fcgi
DAEMON_OPTS=" -a 127.0.0.1 -p 9999 -F 4 -u user -U user -P $PID $PREFIX/lib/cgi-bin/mapserv"
start () {
    echo -n $"Starting $NAME "
        exec $DAEMON $DAEMON_OPTS >> /dev/null
        daemon --pidfile $PID
        RETVAL=$?
        echo
    [ $RETVAL -eq 0 ]
}
stop () {
    echo -n $"Stopping $NAME "
        killproc -p $PID
        #make sure all mapservers are closed
        pkill -f lt-mapserv
        RETVAL=$?
        echo
    if [ $RETVAL -eq 0 ] ; then
                rm -f $PID
        fi
}
restart () {
    stop
    start
}
# See how we were called.
case "$1" in
  start)
        start
    ;;
  stop)
        stop
    ;;
  status)
    status lt-mapserv
        RETVAL=$?
        ;;
  restart)
    restart
        ;;
  *)
        echo $"Usage: $0 {start|stop|status|restart}"
        RETVAL=2
        ;;
esac
exit $RETVAL
{% endhighlight %}

_*I'm not using Linux so not sure what line what do, but want to believe that it configured fine._

Make this file executable:

{% highlight xml %}
chmod +x /etc/init.d/mapserv
{% endhighlight %}

nginx config
---------------------

Create file _/etc/nginx/sites-enabled/your-file-name_ with:

{% highlight xml %}
server {
    ##another server config
    listen   80;
    server_name  your_server.name www.mapserver.your_server.name;
    #MapServer
        location / {
                fastcgi_pass   127.0.0.1:9999;
                fastcgi_index  mapserv?*;
                fastcgi_param  SCRIPT_FILENAME  /usr/lib/cgi-bin/mapserv$fastcgi_script_name;
                include fastcgi_params;

    }
}
{% endhighlight %}

I had made all thing on localhost. So not need to create new file, but just add lines into _/etc/nginx/sites-enabled/default_:

{% highlight xml %}
location /map/ {
	fastcgi_pass   127.0.0.1:9999;
	fastcgi_index  mapserv?*;
	fastcgi_param  SCRIPT_FILENAME  /usr/lib/cgi-bin/mapserv$fastcgi_script_name;
	include fastcgi_params;
}
{% endhighlight %}

We done with configurations and now can start Mapserver and nginx :

{% highlight xml %}
service mapserv start
/etc/init.d/nginx start
{% endhighlight %}

Lets check what we done. Go to _localhost/map/_ and if you do all right you'll see:

{% highlight xml %}
No query information to decode. QUERY_STRING is set, but empty.
{% endhighlight %}

Create map
---------------------

Now lets do our first map. I take some example data files here: http://gis-lab.info/other/mapserver-begin-example.zip. 
Extract it everywhere you want, for example here /home/my-user/example.

MapServer have no user friendly administration panel like Geoserver's one.
To make a map we need to create *.map files that contains services, map, layers, styles, etc. 
In zip already contain some. We'll use simple one - _polt.map_:

{% highlight xml %}
MAP
  IMAGETYPE      GIF
  EXTENT         34.59 49.58 34.63 49.6
  SIZE           400 300
  SHAPEPATH      "/home/user/example/shp/"
  IMAGECOLOR     255 255 255

  LAYER
    NAME         veget
    DATA         Poltava10_Vegetation_region
    STATUS       ON
    TYPE         POLYGON
    CLASS
      NAME       "Растительность"
      STYLE
        COLOR        232 232 232
        OUTLINECOLOR 32 32 32
      END
    END
  END 

END
{% endhighlight %}

_*I'd removed comments from file_

This map contains only one layer _veget_. If you unzip text data in another place, please change _SHAPEPATH_ parameter to actual.

Lets see what we got:

{% highlight xml %}
http://localhost/map/?map=/home/user/example/polt.map&layer=veget&mode=map
{% endhighlight %}

And here is expected result:

![Simple Mapserver Map](/blog/img/2013-06-26/mapserver-begin-01.gif "Simple Mapserver Map")

Conclusion
---------------------

Thanks for [Yodeski Rodríguez Álvarez](https://github.com/yodeski) for nginx and FastCGI configuration files.
Map creating part have taken from [gis-lab.ru](http://gis-lab.info/qa/mapserver-begin.html).


