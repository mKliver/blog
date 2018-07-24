---
title: Labels rotation
---

I needed to create SLD style for lines with labels rotated on angle equals line azimuth in start point of line.
Like this:

![Line rotation](/blog/img/2014-08-24/result-line.jpg "Line rotation")

DB
---------------------

I want to set rotation angle by feature attribute contained this angle. Data stored in PostgresSQL db.

{% highlight sql linenos %}
CREATE TABLE annotations
(
  gid serial NOT NULL,
  annotation character varying(250),
  the_geom geometry,
  angle character varying(20) DEFAULT 0,
  CONSTRAINT annotations_pkey PRIMARY KEY (gid),
  CONSTRAINT enforce_dims_the_geom CHECK (st_ndims(the_geom) = 2),
  CONSTRAINT enforce_srid_the_geom CHECK (st_srid(the_geom) = 4326)
)
WITH (
  OIDS=FALSE
);
ALTER TABLE annotations
  OWNER TO postgres;
{% endhighlight %}

Lets add trigger to calculate angle to each one ne feature:

{% highlight sql linenos %}
CREATE TRIGGER set_angle
  BEFORE INSERT OR UPDATE
  ON annotations
  FOR EACH ROW
  EXECUTE PROCEDURE setangle();

CREATE OR REPLACE FUNCTION setangle()
  RETURNS trigger AS
$BODY$
BEGIN
   NEW.angle := degrees(
                             ST_Azimuth(
                                ST_Transform(ST_StartPoint(NEW.the_geom),900913)
                              , ST_Transform(ST_EndPoint(NEW.the_geom),900913)
                             )
                          ) - 90;
   RETURN NEW;
END
$BODY$
  LANGUAGE plpgsql VOLATILE
  COST 100;
ALTER FUNCTION setangle()
  OWNER TO postgres;
{% endhighlight %}

As you can notice my table __SRID=4326__, but in calculation points transform into SRID=900913. 
It's because my application map projection is __EPSG:900913__. If do not transform point to projection what you will show this table you get difference between line azimuth and angle field value:

![Wrong rotation](/blog/img/2014-08-24/projection-problem.jpg "Wrong rotation")

SLD
---------------------

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<StyledLayerDescriptor version="1.0.0"
 xsi:schemaLocation="http://www.opengis.net/sld StyledLayerDescriptor.xsd"
 xmlns="http://www.opengis.net/sld"
 xmlns:ogc="http://www.opengis.net/ogc"
 xmlns:xlink="http://www.w3.org/1999/xlink"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <NamedLayer>
    <Name>Annotatioans</Name>
    <UserStyle>
      <Title>Annotations</Title>
      <Abstract>Annotations white text</Abstract>
      <FeatureTypeStyle>
      <Rule>        
        <TextSymbolizer>
          <Geometry>
            <ogc:Function name="startPoint">
              <ogc:PropertyName>the_geom</ogc:PropertyName>
            </ogc:Function>
          </Geometry>
          <Label>
            <ogc:PropertyName>annotation</ogc:PropertyName>
          </Label>
           <Font>
             <CssParameter name="font-family">Arial</CssParameter>
             <CssParameter name="font-size"><ogc:PropertyName>fontSize</ogc:PropertyName></CssParameter>
             <CssParameter name="font-style">normal</CssParameter>
             <CssParameter name="font-weight">bold</CssParameter>
           </Font>
          <LabelPlacement>
           <PointPlacement>
             <AnchorPoint>
               <AnchorPointX>0.0</AnchorPointX>
               <AnchorPointY>0.5</AnchorPointY>
             </AnchorPoint>
             <Displacement>
               <DisplacementX>0</DisplacementX>
               <DisplacementY>5</DisplacementY>
             </Displacement>
             <Rotation>
              <ogc:PropertyName>rotationAngle</ogc:PropertyName>
             </Rotation>
           </PointPlacement>
          </LabelPlacement>
          <Halo>
            <Radius>1</Radius>
            <Fill>
              <CssParameter name="fill">#000000</CssParameter>
            </Fill>
          </Halo>
          <Fill>
            <CssParameter name="fill">#FFFFFF</CssParameter>
          </Fill>
          <VendorOption name="spaceAround">10</VendorOption>
          <VendorOption name="group">yes</VendorOption>
        </TextSymbolizer>
        <LineSymbolizer>
          <Stroke>
            <CssParameter name="stroke">#000000</CssParameter>
          </Stroke>
        </LineSymbolizer>
      </Rule>
    </FeatureTypeStyle>
  </UserStyle>
 </NamedLayer>
</StyledLayerDescriptor>
{% endhighlight %}

Simple style. Here is _Rotaion_ block it is a main part of this.

 
