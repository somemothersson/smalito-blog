---
title: "Batch Geocoding"
date: "2019-09-05T22:12:03.284Z"
description: "A less than perfect approach to solving a geocoding dilemma"
---

##   Here's a hacky way to get geocodes for places 
If you're working with multiple locations in Google Maps, you need to use geocodeing aka geocoordinates aka lattitude and longitude. Now don't get all bad lattitude on me, I've got something for you if what you thought was gonna be simple has now turned into a longitude and drawn out process.

You can fight through Google's vast documentation, which when you do figure out; will net you more data than you need for a simple project; or waste your time on [latlong.net]( https://www.latlong.net/) to prepopulate the coordinates one by one. 

My plan was to get the geocoordinates formatted into a JSON objected designed for my specific purpose. I would then populate this to my mongo database, where I would use it to interface with requests from census data on a remote server that together would allow me to have data formated for heatmap generation via google's API.

### You're gonna need:
* A Mapquest API Key [Mapquest Developer](https://developer.mapquest.com)
* NodeGeocoder NPM Package [NodeGeocoder](https://www.npmjs.com/package/node-geocoder)



### NodeGeocoder Config
Let's get our base config setup for NodeGeocoder. Im using Mapquest because I had an API setup for a previous project.
```javascript
// Yeah, it's required
const NodeGeocoder = require("node-geocoder");

// Setup your options for NodeGeocoder
const options = {
  provider: "mapquest",

  // Optional depending on the providers
  httpAdapter: "request", // Default
  apiKey: "G3Tyur0wnk3yy4j3rk", // for Mapquest, OpenCage, Google Premier
  formatter: null // 'gpx', 'string', ...
};

const geocoder = NodeGeocoder(options);


```
### Preparing our data
In my instance, I had 78 neighborhood names in the remote census API and an "areaId", that `they` made up, which I needed to interface with. Being that array indexes start at 0, and my array needed to have it's first entry at 1. I took the dumbguy approach and just added the first entry twice, which I would then delete after the data was populated. I know its a bit hamfisted, but I was pressed for time, gimme a break *jeeze*.

#### SideBar: Format place and state in a seperate file
If you're using neighborhoods like me, you need to include the city and state. Neighborhood names are too vauge. I preprocessed this by jamming a quick array map in another file with areaArray to add my city and state



```javascript
const areaArray = [
    "Rogers Park", 
    "Rogers Park", 
    "West Ridge", 
    "Uptown"];
const cityState = " Chicago, IL"

const addCityState = areaArray.map(neighborhood => {
    return neighborhood + cityState
})
console.log(addCityState)
```


*Use this in your areaArray*
```javascript
[ 'Rogers Park Chicago, IL',
  'Rogers Park Chicago, IL',
  'West Ridge Chicago, IL',
  'Uptown Chicago, IL' ]
```

### Now for the Spicy Meatball
Lets put this all together and get what we're after! 

```javascript
//Here is our array which will actally be used to get geocoordinates
const areaNameArray = [
  "Rogers Park Chicago,	IL",
  "Rogers Park Chicago,	IL",
  "West Ridge Chicago, IL",
  "Uptown Chicago,	IL"
];

//You may not need this, I did it just for organization and future troubleshooting potential
const areaArray = [
  "Rogers Park", 
  "Rogers Park", 
  "West Ridge", 
  "Uptown"];

// Execute geocoder with our areaNameArray
geocoder.batchGeocode(areaNameArray, (err, results) => {
  for (i = 0; i < results.length; i++) {
    console.log(
      `{"db_area":"${i}", "db_area_name":"${areaArray[i]}", "lat":"${results[i].value[0].latitude}", "lng":"${results[i].value[0].longitude}"},`
    );
  }
});

```


`Output:`
Properly formatted JSON Object I can use with Google MAPS, can I get a Hell YEAH!? 
```json
{"db_area":"0", "db_area_name":"Rogers Park", "lat":"42.010531", "lng":"-87.670748"},
{"db_area":"1", "db_area_name":"Rogers Park", "lat":"42.010531", "lng":"-87.670748"},
{"db_area":"2", "db_area_name":"West Ridge", "lat":"42.001567", "lng":"-87.695137"},
{"db_area":"3", "db_area_name":"Uptown", "lat":"41.966063", "lng":"-87.656105"},
```

#### PHEW
Now we're cooking with gas. We got output in our console `EXACTLY` the way we want it. So all I did was paste this into Postman and Posted it to my appropriate route. Don't forget to delete that first guy @0 before you upload, and *Ala Kazam Ala Kazoo*

Hope this helps someone, someplace, at sometime.




