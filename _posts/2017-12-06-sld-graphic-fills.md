---
title: SLD Graphic fills
tags: [sld]
---
This style contains fills with two size different graphics. 
I'd founded a style:
{% highlight xml %}
<PolygonSymbolizer>
	<Fill>
		<GraphicFill>
			<Graphic>
				<Mark>
					<WellKnownName>circle</WellKnownName>
					<Stroke>
						<CssParameter name="stroke">#000000</CssParameter>
						<CssParameter name="stroke-width">1</CssParameter>
					</Stroke>
				</Mark>
				<Size>8</Size>
			</Graphic>
		</GraphicFill>
	</Fill>
	<sld:VendorOption name="graphic-margin">18 18 4 4</sld:VendorOption>
</PolygonSymbolizer>
<PolygonSymbolizer>
	<Fill>
		<GraphicFill>
			<Graphic>
				<Mark>
					<WellKnownName>circle</WellKnownName>
					<Fill>
						<CssParameter name="fill">#000000</CssParameter>
					</Fill>
				</Mark>
				<Size>6</Size>
			</Graphic>
		</GraphicFill>
	</Fill>
	<sld:VendorOption name="graphic-margin">3 3 21 21</sld:VendorOption>
</PolygonSymbolizer>

{% endhighlight %}

Here is two _PolygonSymbolizer_ with different circles inside.  
Main part of style is _VendorOption name="graphic-margin"_ tag. Its allows add space around graphics.
It's very clear how its works when i sets same margin around one graphic, but when i have two graphics
i needed to set different space by each graphic's sides.

Here is schema explain margin spacing:
![Schema](/blog/img/2017-12-06/2017-12-06_1.jpg "Schema")

So how you see, i got a same length for each side of square for each figure.

{% highlight c++ %}
	For yellow circle: 4+8+18=30
	For red circle: 3+6+21=30
{% endhighlight %}

And the result:
![Result style](/blog/img/2017-12-06/2017-12-06_2.png "Result style")

