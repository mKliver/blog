---
title: SLD Dash Line
---

Simple dash line with mark in centre of each hatch.

![Schema](/blog/img/2018-07-24/2018-07-24-2.jpg "Schema")

A style:
{% highlight xml linenos %}

<FeatureTypeStyle>
 <Rule>
   <LineSymbolizer>
	 <Stroke>
	   <CssParameter name="stroke">#0000FF</CssParameter>
	   <CssParameter name="stroke-width">1</CssParameter>
	   <CssParameter name="stroke-dasharray">40 10</CssParameter>
	 </Stroke>
   </LineSymbolizer>
   <LineSymbolizer>
	 <Stroke>
	   <GraphicStroke>
		 <Graphic>
		   <Mark>
			 <WellKnownName>circle</WellKnownName>
			 <Stroke>
			   <CssParameter name="stroke">#000033</CssParameter>
			   <CssParameter name="stroke-width">5</CssParameter>
			 </Stroke>
		   </Mark>
		   <Size>5</Size>
		 </Graphic>
	   </GraphicStroke>
	   <CssParameter name="stroke-dasharray">5 45</CssParameter>
	   <CssParameter name="stroke-dashoffset">32.5</CssParameter>
	 </Stroke>
   </LineSymbolizer>
 </Rule>
</FeatureTypeStyle>

{% endhighlight %}

Here is schema explain stype values:
![Schema](/blog/img/2018-07-24/2018-07-24-1.jpg "Schema")

{% highlight xml %}

S1, D1 - Stroke and Dash of line
S2, D2 - Stroke and Dash of mark
offset - stroke-dashoffset paremeter value

S1, D1, S2 - is known values so lets find D2 and offset.

D2 = 2(S1/2 - S2/2)+D1

offset = S1/2 + S2/2 + D1

{% endhighlight %}

And solve it with values from schema

{% highlight xml %}

S1 =40
D1 =10
D2 = 5

D2 = 2(40/2 - 5/2)+10

offset = 20 + 5 + 10

{% endhighlight %}