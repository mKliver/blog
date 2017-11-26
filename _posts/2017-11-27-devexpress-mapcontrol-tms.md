---
title: How to show TMS layer in DevExpress Mapcontrol.
---
By default DevExpress Mapcontrol works only with XYZ scheme. And has no options to work with layers in TMS scheme. So we need to write little bit of code.
My solution based on this [blog post](https://dx-developers.com/2015/04/29/using-a-custom-tile-provider-with-the-xtramap-control/). 

At first create a provider class:
{% highlight c# linenos %}

sealed public class TMSProvider : MapDataProviderBase
{
	#region Private members

	/// <summary>
	/// Map project
	/// </summary>
	private readonly SphericalMercatorProjection _Projection = new SphericalMercatorProjection();

	#endregion

	#region Public accessors

	/// <summary>
	/// Gets the Projection for this tile provider
	/// </summary>
	public override ProjectionBase Projection
	{
		get { return _Projection; }
	}

	/// <summary>
	/// Returns a size struct representing the base size of this provider, in pixels
	/// </summary>
	protected override Size BaseSizeInPixels
	{
		get { return new Size(Convert.ToInt32(TMSSource._TileSize * 2), Convert.ToInt32(TMSSource._TileSize * 2)); }
	}

	#endregion


	/// <summary>
	/// Initializes a new instance of the HereMapTileProvider class
	/// </summary>
	public TMSProvider()
	{
		TileSource = new TMSSource(this);
	}


	/// <summary>
	/// Returns the pixel size of this map for the provided zoom level
	/// </summary>
	/// <param name="zoomLevel">Current zoom level</param>
	/// <returns></returns>
	public override MapSize GetMapSizeInPixels(double zoomLevel)
	{
		double imageSize;
		imageSize = TMSSource.CalculateTotalImageSize(zoomLevel);

		return new MapSize(imageSize, imageSize);

	}
}

{% endhighlight %}

Second source class:
{% highlight c# linenos %}

sealed public class TMSSource : MapTileSourceBase
{
	/// <summary>
	/// Tile image size (in pixels)
	/// </summary>
	internal const int _TileSize = 256;

	/// <summary>
	/// Maximum zoom level
	/// </summary>
	internal const int _MaxZoomLevel = 20;

	/// <summary>
	/// Intializes a new instance of the HereMapTileSource class
	/// </summary>
	/// <param name="cacheOptionsProvider"></param>
	public TMSSource(ICacheOptionsProvider cacheOptionsProvider) :
		base((int)CalculateTotalImageSize(_MaxZoomLevel), (int)CalculateTotalImageSize(_MaxZoomLevel), 
		_TileSize, _TileSize, cacheOptionsProvider)
	{

	}

	/// <summary>
	/// Returns the total image size for the provided zoom level
	/// </summary>
	/// <param name="zoomLevel"></param>
	/// <returns></returns>
	internal static double CalculateTotalImageSize(double zoomLevel)
	{
		if (zoomLevel < 1.0)
			return zoomLevel * _TileSize * 2;

		return Math.Pow(2.0, zoomLevel) * _TileSize;

	}

	/// <summary>
	/// Returns a Uri used to fetch tiles for the provided zoom level and co-ordinates
	/// </summary>
	/// <param name="zoomLevel">Current zoom level</param>
	/// <param name="tilePositionX">Current tile x co-ordinate</param>
	/// <param name="tilePositionY">Current tile y co-ordinate</param>
	/// <returns></returns>
	public override Uri GetTileByZoomLevel(int zoomLevel, int tilePositionX, int tilePositionY)
	{

		string Url = String.Format(@"http://localhost:8080/tiles/{0}/{1}/{2}.png",
					zoomLevel                         
					tilePositionX,
					Math.Pow(2, zoomLevel)- tilePositionY-1);

		if (zoomLevel <= _MaxZoomLevel)
			return new Uri(Url);
		else
			return null;

	}
}

{% endhighlight %}

Most interesting moment in **GetTileByZoomLevel** method - y coordinate calculation. Here is formula:
{% highlight ruby %}

y = (2^z) - y - 1

{% endhighlight %}

More information about tile schemes [here](https://gist.github.com/tmcw/4954720). 

And finally usage. Put MapControl on the form:
{% highlight c# linenos %}

public Form1()
{
	InitializeComponent();

	ImageTilesLayer imageTilesLayer = new ImageTilesLayer();
	mapControl1.Layers.Add(imageTilesLayer);
	imageTilesLayer.DataProvider = new TMSProvider();
}

{% endhighlight %}
