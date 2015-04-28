# turf-lrs
Linear referencing modules modeled after the Turf.js library of modules

This little library will be built to work with a couple custom geoJSON feature types, linestring-m and lrs-event.  Linestring-m is a valid geoJSON linestring with 4-dimensional coordinates included in the geometry.  It will also have a dimInfo property describing the four dimensions like `['x','y','z','m']` (this is the default configuration, all tools built to take a linestring-m will need to interrogate the dimInfo property before proceeding to make sure where the 'm' coordinate is, if the linestring doesn't have a dimInfo array then it is assumed that 'm' is the fourth member of the coordinate array and we don't guarantee good results if this is not the case.  Linestring-m features can and should contain the optional routeId property.  This helps to find the correct linestring-m feature to use during runtime from feature collections. The lrs-event feature type is not part of the geoJSON spec, but is used to store information and interact with linestring-m features to produce valid geoJSON for rendering.  Lrs-event objects will take the following form:
```
{
	type:'Feature',
    geometry:{
    	type:'Event',
        coordinates:[from-m-value,to-m-value,offset]
    },
    properties:{
    	routeId:'stringOrNumberIdValue'
    }
}
```
Lrs-events are feature objects with a geometry type of `'Event'`.  The coordinates array for the Event type differes from that used in the traditional geoJSON geometry object.  Each lrs-event feature has at least one coordinate, the from-m-value.  to-m-value is used to describe a line segment, or if using the offset coordinate, the from and the to-m-values should be set to the same number.

`turf-lrs-extract-line` -> take a linestring-m and start and end m values [or a lrs-event object] and return the line geometry for that portion of the routed line
	
    * if start-m < line-begin-m then start-m = line-begin-m
    * if end-m > line-end-m then end-m = line-end-m
    * returned line inherits all coordinate values from the source linestring-m
    * returned line will have an empty properties collection unless properties are passed in as a parameter

`turf-lrs-extract-point` -> take a measure value [or a lrs-event object] and an optional offset value to get the point geometry back for that location on the line or to the side of the line
	
    * if m value is not on the line return an empty geometry
    * returned point will have all coordinates from the source linestring, interpolated if the point lies between vertices
    * returned point will have an empty properties collection unless properties are passed in as a parameter or with the lrs-event object

`turf-lrs-begin-m` -> take a linestring-m and return the m value of the start of the line

`turf-lrs-end-m` -> take a linestring-m and return the m value of the end of the line

`turf-lrs-locate-point` -> take a linestring-m and a point and return the measure value and offset of the point relative to the line
	
    * should take a tolerance parameter if we want to snap locations close to the line onto the line, should probably have a default value of 2 meters or something
    * return a lrs-event object that inherits the properties of the source point    

`turf-lrs-locate-line` -> take a linestring-m and another linestring and return an array of m values where the lines intersect
	
    * should use the same default tolerance as the turf-lrs-locate-point module
    * should return a feature collection of lrs-events where the line crosses the linestring-m more than once, or a single lrs-event if it only intersects once.  each lrs-event returned will inherit the properties of the input line being located

`turf-lrs-calibrate` -> take a linestring and return a linestring-m with measure values corresponding to the linear distance between vertices on the line
