---
title: Some SLD styles
---

Here is some SLD examples what i'd used.

###Empty star with default marks

![Empty star](/blog/img/2014-06-14/empty-star.jpg "Empty star")

{% highlight xml linenos %}
<PointSymbolizer>
	<Graphic>
		<Mark>
			<WellKnownName>star</WellKnownName>
			<Fill>
				<CssParameter name="fill">#000000</CssParameter>
			</Fill>
		</Mark>
		<Size>18</Size>
	</Graphic>
</PointSymbolizer>
<PointSymbolizer>
	<Graphic>
		<Mark>
			<WellKnownName>star</WellKnownName>
			<Fill>
				<CssParameter name="fill">#FF0000</CssParameter>
			</Fill>
		</Mark>
		<Size>16</Size>
	</Graphic>
</PointSymbolizer>
<PointSymbolizer>
	<Graphic>
		<Mark>
			<WellKnownName>star</WellKnownName>
			<Fill>
				<CssParameter name="fill">#FFFFFF</CssParameter>
			</Fill>
		</Mark>
		<Size>10</Size>
	</Graphic>
</PointSymbolizer>
{% endhighlight %}

All three point symbolizers pretty same, just difference in size and colour. 
Just make sure that you put a smallest mark on the top and biggest on down.

###Railroad

![Railroad](/blog/img/2014-06-14/rail-road.jpg "Railroad")

{% highlight xml %}
<LineSymbolizer>
	 <Stroke>
	  <CssParameter name="stroke">#00FF00</CssParameter>
	 </Stroke>
	</LineSymbolizer>
<LineSymbolizer>
	<Stroke>
		<GraphicStroke>
			<Graphic>
				<Mark>
					<WellKnownName>shape://vertline</WellKnownName>
					<Stroke>
						<CssParameter name="stroke">#808080</CssParameter>
						<CssParameter name="stroke-width">1</CssParameter>
					</Stroke>
				</Mark>
				<Size>8</Size>
			</Graphic>
		</GraphicStroke>
		<CssParameter name="stroke-dasharray">10 30</CssParameter>
	</Stroke>
</LineSymbolizer>
<LineSymbolizer>
	<Stroke>
		<GraphicStroke>
			<Graphic>
				<Mark>
					<WellKnownName>shape://vertline</WellKnownName>
					<Stroke>
						<CssParameter name="stroke">#808080</CssParameter>
						<CssParameter name="stroke-width">1</CssParameter>
					</Stroke>
				</Mark>
				<Size>8</Size>
			</Graphic>
		</GraphicStroke>
		<CssParameter name="stroke-dasharray">10 30</CssParameter>
		<CssParameter name="stroke-dashoffset">5</CssParameter>
	</Stroke>
</LineSymbolizer>
{% endhighlight %}

And little bit complex.

![Railroad](/blog/img/2014-06-14/rail-road-2.jpg "Railroad")

{% highlight xml %}
<LineSymbolizer>
	 <Stroke>
	  <CssParameter name="stroke">#00FF00</CssParameter>
	 </Stroke>
	</LineSymbolizer>
<LineSymbolizer>
	<Stroke>
		<GraphicStroke>
			<Graphic>
				<Mark>
					<WellKnownName>shape://vertline</WellKnownName>
					<Stroke>
						<CssParameter name="stroke">#808080</CssParameter>
						<CssParameter name="stroke-width">1</CssParameter>
					</Stroke>
				</Mark>
				<Size>8</Size>
			</Graphic>
		</GraphicStroke>
		<CssParameter name="stroke-dasharray">10 30</CssParameter>
	</Stroke>
</LineSymbolizer>
<LineSymbolizer>
	<Stroke>
		<GraphicStroke>
			<Graphic>
				<Mark>
					<WellKnownName>shape://vertline</WellKnownName>
					<Stroke>
						<CssParameter name="stroke">#808080</CssParameter>
						<CssParameter name="stroke-width">1</CssParameter>
					</Stroke>
				</Mark>
				<Size>8</Size>
			</Graphic>
		</GraphicStroke>
		<CssParameter name="stroke-dasharray">10 30</CssParameter>
		<CssParameter name="stroke-dashoffset">5</CssParameter>
	</Stroke>
</LineSymbolizer>
<LineSymbolizer>
	<Stroke>
		<GraphicStroke>
			<Graphic>
				<Mark>
					<WellKnownName>shape://vertline</WellKnownName>
					<Stroke>
						<CssParameter name="stroke">#808080</CssParameter>
						<CssParameter name="stroke-width">1</CssParameter>
					</Stroke>
				</Mark>
				<Size>8</Size>
			</Graphic>
		</GraphicStroke>
		<CssParameter name="stroke-dasharray">10 30</CssParameter>
		<CssParameter name="stroke-dashoffset">10</CssParameter>
	</Stroke>
</LineSymbolizer>
{% endhighlight %}

You can continue this just by adding next _LineSymbolizer_ with same parameters, but increase _stroke-dashoffset_.

 
