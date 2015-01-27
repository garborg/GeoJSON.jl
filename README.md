# GeoJSON.jl

[![Build Status](https://travis-ci.org/yeesian/GeoJSON.jl.svg)](https://travis-ci.org/yeesian/GeoJSON.jl)
[![Coverage Status](http://img.shields.io/coveralls/yeesian/GeoJSON.jl.svg)](https://coveralls.io/r/yeesian/GeoJSON.jl)

This library is developed independently of, but is heavily influenced in design by the [python-geojson](https://github.com/frewsxcv/python-geojson) package. It contains:

- Functions for encoding and decoding GeoJSON formatted data
- a type hierarchy (according to the [GeoJSON specification](http://geojson.org/geojson-spec.html))
- An implementation of the [__geo_interface__](https://gist.github.com/sgillies/2217756), a GeoJSON-like protocol for geo-spatial (GIS) vector data.

## Installation

`GeoJSON.jl` is not a listed package (yet). Heres what you're going to need to do to install it:

```julia
# Install GeoJSON direct from this repository to your Julia package directory
Pkg.clone("https://github.com/yeesian/GeoJSON.jl.git")
# Running Pkg.update() will always give you the freshest version of GeoJSON
Pkg.test("GeoJSON")
# Double-check that it works
```

## Basic Usage
Although we introduce types for representing GeoJSON objects, it works in tandem with the [JSON.jl](https://github.com/JuliaLang/JSON.jl) package, for parsing and printing objects. Here are some examples of its functionality:

- Parses a JSON String or IO stream into a GeoJSON object
```julia
julia> using GeoJSON
julia> osm_buildings = """{
                "type": "FeatureCollection",
                "features": [{
                  "type": "Feature",
                  "geometry": {
                    "type": "Polygon",
                    "coordinates": [
                      [
                        [13.42634, 52.49533],
                        [13.42660, 52.49524],
                        [13.42619, 52.49483],
                        [13.42583, 52.49495],
                        [13.42590, 52.49501],
                        [13.42611, 52.49494],
                        [13.42640, 52.49525],
                        [13.42630, 52.49529],
                        [13.42634, 52.49533]
                      ]
                    ]
                  },
                  "properties": {
                    "color": "rgb(255,200,150)",
                    "height": 150
                  }
                }]
              }"""
julia> buildings = GeoJSON.parse(osm_buildings) # GeoJSON.parse -- string or stream to AbstractGeoJSON types
FeatureCollection([Feature(Polygon({{{13.42634,52.49533},{13.4266,52.49524},{13.42619,52.49483},{13.42583,52.49495},{13.4259,52.49501},{13.42611,52.49494},{13.4264,52.49525},{13.4263,52.49529},{13.42634,52.49533}}},#undef,#undef),["height"=>150,"color"=>"rgb(255,200,150)"],#undef,#undef,#undef)],#undef,#undef)
```

- Transforms a GeoJSON object into a nested Array or Dict

```julia
julia> GeoJSON.geojson2dict(buildings) # geojson2dict -- AbstractGeoJSON to Dict/Array-representation
Dict{String,Any} with 2 entries:
  "features" => [["geometry"=>["coordinates"=>{{{13.42634,52.49533},{13.4266,52.49524},{13.42619,52.49483},{13.42583,52.49495},{13.4259,52.49…
  "type"     => "FeatureCollection"

julia> JSON.parse(osm_buildings) # should be comparable (if not the same)
Dict{String,Any} with 2 entries:
  "features" => {["geometry"=>["coordinates"=>{{{13.42634,52.49533},{13.4266,52.49524},{13.42619,52.49483},{13.42583,52.49495},{13.4259,52.49…
  "type"     => "FeatureCollection"
```

- Transforms from a nested Array/Dict to a GeoJSON object

```julia
julia> GeoJSON.dict2geojson(GeoJSON.geojson2dict(buildings))
FeatureCollection([Feature(Polygon({{{13.42634,52.49533},{13.4266,52.49524},{13.42619,52.49483},{13.42583,52.49495},{13.4259,52.49501},{13.42611,52.49494},{13.4264,52.49525},{13.4263,52.49529},{13.42634,52.49533}}},#undef,#undef),["height"=>150,"color"=>"rgb(255,200,150)"],#undef,#undef,#undef)],#undef,#undef)

julia> GeoJSON.parse(osm_buildings) # the original object (for comparison)
FeatureCollection([Feature(Polygon({{{13.42634,52.49533},{13.4266,52.49524},{13.42619,52.49483},{13.42583,52.49495},{13.4259,52.49501},{13.42611,52.49494},{13.4264,52.49525},{13.4263,52.49529},{13.42634,52.49533}}},#undef,#undef),["height"=>150,"color"=>"rgb(255,200,150)"],#undef,#undef,#undef)],#undef,#undef)
```

- Returns a compact JSON representation as a String

```julia
julia> geojson(buildings) # AbstractGeoJSON to a string
"{\"features\":[{\"geometry\":{\"coordinates\":[[[13.42634,52.49533],[13.4266,52.49524],[13.42619,52.49483],[13.42583,52.49495],[13.4259,52.49501],[13.42611,52.49494],[13.4264,52.49525],[13.4263,52.49529],[13.42634,52.49533]]],\"type\":\"Polygon\"},\"properties\":{\"height\":150,\"color\":\"rgb(255,200,150)\"},\"type\":\"Feature\"}],\"type\":\"FeatureCollection\"}"

julia> JSON.json(JSON.parse(osm_buildings)) # compared with the JSON parser
"{\"features\":[{\"geometry\":{\"coordinates\":[[[13.42634,52.49533],[13.4266,52.49524],[13.42619,52.49483],[13.42583,52.49495],[13.4259,52.49501],[13.42611,52.49494],[13.4264,52.49525],[13.4263,52.49529],[13.42634,52.49533]]],\"type\":\"Polygon\"},\"properties\":{\"height\":150,\"color\":\"rgb(255,200,150)\"},\"type\":\"Feature\"}],\"type\":\"FeatureCollection\"}"
```

- Writes a compact (no extra whitespace or identation) JSON representation to the supplied IO.

```julia
GeoJSON.print(io::IO, obj::AbstractGeoJSON)
```

## GeoJSON Objects
This library implements the following [GeoJSON Objects](http://www.geojson.org/geojson-spec.html#geojson-objects) described in The GeoJSON Format Specification.

- `Geometry <: AbstractGeoJSON`
  - `abstract Geometry`
  - `Point <: Geometry`
  - `MultiPoint <: Geometry`
  - `LineString <: Geometry`
  - `MultiLineString <: Geometry`
  - `Polygon <: Geometry`
  - `MultiPolygon <: Geometry`
  - `GeometryCollection <: Geometry`
- `Feature <: AbstractGeoJSON`
- `FeatureCollection <: AbstractGeoJSON`

The following methods are implemented for all AbstractGeoJSON objects:
```julia
hasbbox(obj::AbstractGeoJSON) # returns true if obj has a "bbox" key
hascrs(obj::AbstractGeoJSON) # returns true if obj has a "crs" key
bbox(obj::AbstractGeoJSON) # returns the boundingbox of obj
crs(obj::AbstractGeoJSON) # returns the coordinate reference system
```
In addition, the `Feature` object also implements ```has_id(obj::Feature)```

### GeoJSON Attributes (GEO_Interface)
In accordance with the [GeoJSON format](http://geojson.org/geojson-spec.html) (and the [__geo_interface__](https://gist.github.com/sgillies/2217756)), the following methods are implemented for each of the GeoJSON objects:
```julia
# GeoJSON             (methods,)
# --------------------------------------------
((MultiPolygon,       (coordinates,)),
 (Polygon,            (coordinates,)),
 (MultiLineString,    (coordinates,)),
 (LineString,         (coordinates,)),
 (MultiPoint,         (coordinates,)),
 (Point,              (coordinates,)),
 (GeometryCollection, (geometries,)),
 (Feature,            (geometry, properties)),
 (FeatureCollection,  (features,)))
```
