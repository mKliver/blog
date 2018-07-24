---
title: SLD Dash Line
---
Simple dash line with mark in centre of each hatch.

![Schema](/blog/img/2018-07-28/2018-07-28-2.jpg "Schema")

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
![Schema](/blog/img/2018-07-28/2018-07-28-1.jpg "Schema")

{% highlight %}

__S1__, __D1__ - Stroke and Dash of line
__S2__, __D2__ - Stroke and Dash of mark
__offset__ - _stroke-dashoffset_ paremeter value

__S1__, __D1__, __S2__ - is known values so lets find __D2__ and __offset__.

__D2__ = 2(__S1__/2 - __S2__/2)+__D1__

__offset__ = S1/2 + S2/2 + D1

{% endhighlight %}

And solve it with values from schema

{% highlight %}

__S1__=40
__D1__=10
__D2__ = 5

__D2__ = 2(40/2 - 5/2)+10

__offset__ = 20 + 5 + 10

{% endhighlight %}