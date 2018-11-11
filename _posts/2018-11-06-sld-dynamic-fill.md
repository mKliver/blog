---
title: SLD dynamic fill
tag: sld
---

Simple example of dynamic sld style. But it is not a trully dynamic style, it is just a tricky Fill.<br>
Based on _Categorize_ function and CQL filter. At first i choose property, what i want to base to, and default value(literally this is a maximum property value).<br>
After based on this value create intervals for coloured features in different colours - 1200-*, 900-1200, 700-900 etc.
Now divide interval's low borders on max value i got a coefficient what compares in _Categorize_ function.

A style:
{% highlight xml linenos %}

<se:FeatureTypeStyle>
<se:Rule>
  <se:Name> >1201 </se:Name>
  <se:Description>
	<se:Title> >1201 </se:Title>
  </se:Description>
  <se:PolygonSymbolizer>
	   <se:Fill>
		 <se:SvgParameter name="fill">
		   <ogc:Function name="Categorize">
			 <!-- Value to transform -->
			 <ogc:Div>
			   <ogc:Function name="env">
					<ogc:Literal>high</ogc:Literal>
					<ogc:Literal>1200</ogc:Literal>
				</ogc:Function>
				<ogc:PropertyName>count</ogc:PropertyName>
			 </ogc:Div>
			 
			 <!--1200-*-->
			 <ogc:Literal>#ff0000</ogc:Literal> 
			 <ogc:Literal>1</ogc:Literal>
			  <!--900-1200-->
			 <ogc:Literal>#8b3058</ogc:Literal>
			 <ogc:Literal>1.33</ogc:Literal>
			 <!--700-900-->					 
			 <ogc:Literal>#ad466c</ogc:Literal>
			 <ogc:Literal>1.71</ogc:Literal>
			 <!--500-700-->
			 <ogc:Literal>#cc607d</ogc:Literal>
			 <ogc:Literal>2.4</ogc:Literal>
			 <!--300-500-->
			 <ogc:Literal>#3b3434</ogc:Literal>
			 <ogc:Literal>3.99</ogc:Literal>
			 <!--100-300-->
			 <ogc:Literal>#f4a3a8</ogc:Literal>
			 <ogc:Literal>11.89</ogc:Literal>
			 <!--*-100-->
			 <ogc:Literal>#0000ff</ogc:Literal>
			 
		   </ogc:Function>
		 </se:SvgParameter>
	   </se:Fill>
	   <se:Stroke>
		  <se:SvgParameter name="stroke">#3b3434</se:SvgParameter>
		  <se:SvgParameter name="stroke-opacity">0.5</se:SvgParameter>
		  <se:SvgParameter name="stroke-width">1</se:SvgParameter>
		  <se:SvgParameter name="stroke-linejoin">mitre</se:SvgParameter>
		</se:Stroke>
	</se:PolygonSymbolizer>
</se:Rule> 
</se:FeatureTypeStyle>

{% endhighlight %}

With default value layer looks:

![Default style](/blog/img/2018-11-06/2018-11-06-default.jpg "Default style")

Now add CQL filter:

{% highlight xml %}
&env=high:900&CQL_FILTER=count<%3D900
{% endhighlight %}

It's mean that features with _count_ property less that 900 will shows on map. And _Categorize_ function will use _high=900_. So result:

![900](/blog/img/2018-11-06/2018-11-06-900.jpg "900")

{% highlight xml %}
&env=high:100&CQL_FILTER=count<%3D100
{% endhighlight %}

![100](/blog/img/2018-11-06/2018-11-06-100.jpg "100")

Ofcouse this style will works with only syntetic data.