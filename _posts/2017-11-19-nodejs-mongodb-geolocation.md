---
layout: post
title: Working with Geospatial support in MongoDB with NodeJS
---

MongoDB’s geospatial indexing allows you to efficiently execute spatial queries on a collection that contains geospatial shapes and points. This tutorial will briefly introduce the concepts of geospatial indexes, and then demonstrate their use with $geoWithin, $geoIntersects, and geoNear.

At my current project I’m working on requires storage of and queries on Geospatial data. We are using MongoDB, which has good support for Geospatial data, at least good enough for my needs. This post walks through the basics of how to work with mongodb geospatial data in `Nodejs` application.
We are using MongoDB `3.4`. Mongo has some very awesome and useful function for doing spatial operation like `$geoNear` `$near` `$nearSphere` `$centerSphere` etc. All of them has some restriction like they don't support shading. Here I will use `$near` with nodejs example.

MongoDB supports storage of geospatial types, represented as `GeoJSON` objects, like `Point`, `LineString` and `Polygon`. I will use `Point` one in this post.

**Query options**

* Inclusion: Whether locations are included in a polygon
* Intersection: Whether locations intersect with a specified geometry
* Proximity: Querying for points nearest other points

**Index options**

* 2d : Calculations are done based on flat geometry
* 2dsphere : Calculations are done based on spherical geometry

As you can imagine, 2dsphere is more accurate, especially for points that are further apart. In this example, I’m using a 2dsphere index, and doing proximity queries.

First I'm doing the Mongo Schema Design with express and assume the model is a `Car` we will do geo spatial query with car location.

 ```
 // CarModel.js

 import mongoose from 'mongoose';
 const Schema = mongoose.Schema;
 const geoSchema = new Schema({
  type: {
    type: String,
    default: 'Point'
  },
  coordinates: {
    type: [Number]
  }
});

const carSchema = new Schema({
  car_id : {
    type : String,
    required : [true, 'Car id is required']
  },
  car_name : {
    type : String
  },
  location : {
    type: geoSchema,
    index: '2dsphere'
  }
},{timestamp : true});

const Car = mongoose.model('Car', carSchema);

export default Car;

```
as you see I'm doing `2dsphere` indexing the location. Now I'm writing the insertion and Queries for geo spatial for finding nearest car.

Inserting car into mongodb
```
const newCar = new Car({
  { car_id: "1234556", location: { type: "Point", coordinates: [90.361006, 23.752726]}}
});

newCar.save().then(car=>{
  // do something
}).catch(err=>{
  // debug
})

```

Ok, Now Lets find nearest car of a point,

```
// Query, distance is in meters
Car.find({location: {$near: {$geometry: {type: "Point", coordinates: [90.366322, 23.755469]}, $maxDistance: 100}}})

```
You can increase or decrease the `maxDistance` as you want.

Cheers :)