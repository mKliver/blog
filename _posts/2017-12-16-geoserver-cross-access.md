---
title: Geoserver Cross domain request
---
I have a Geoserver installed as service and IIS server with my openlayer application.
Every time when i tried send GetFeatureInfo request i got an error:

{% highlight xml linenos %}

Cross-Origin Request Blocked: 
The Same Origin Policy disallows reading the remote resource at
http://localhost:8080/geoserver/robot/ows?service=WFS&version=1.0.0&request=GetFeature&typeName=robot:geofield&outputFormat=aoolication%2Fjson.
(Reason: CORS header 'Access-Control-Allow-Origin' missing).
{% endhighlight %}

Solution was paste into \GeoServer 2.9.0\etc\webdefault.xml this:
{% highlight xml %}
<filter>
    <filter-name>cross-origin</filter-name>
        <filter-class>org.eclipse.jetty.servlets.CrossOriginFilter</filter-class>
    </filter>
    ...
    <filter-mapping>
        <filter-name>cross-origin</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
{% endhighlight %}

When i got Geoserver instaled as application same code works in web.xml.


