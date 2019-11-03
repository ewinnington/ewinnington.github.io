Title: Javascript google maps basics
Published: 24/04/2012
Tags: [Migrated, Google, Javascript, MapsAPI] 
---

Today, I had to embed a google maps with both a fixed KML and one generated from a php script. Here is the webpage source with embedded JavaScript:
```html
<!DOCTYPE html>
<html>
<head>Google Maps Javascript API Test</head>
<style type="text/css">
html, body, #map\_canvas {
margin: 0;
padding: 0;
height: 100%;
}
</style>
<script type="text/javascript" src="https://maps.googleapis.com/maps/api/js?sensor=false"></script>
<script type="text/javascript">
var map;

function initialize() {
var myOptions = { zoom: 11,
center: new google.maps.LatLng(46.045119,7.47757),
mapTypeId: google.maps.MapTypeId.ROADMAP};
map = new google.maps.Map(document.getElementById('map\_canvas'), myOptions);

//Base race overlay from static Website
var ctaLayer = new google.maps.KmlLayer('http:// ... /files/pdg.track.kml');
ctaLayer.setMap(map);

//Patrol Marker
var markerOptions = {
position: new google.maps.LatLng(46.03, 7.47),
map: map,
title:"Derniere position"}

var mark = new google.maps.Marker(markerOptions);

//Infowindow contents
var contentString = '<div id="content">'+
'<div id="patrolNotice">'+
'</div>'+
'<h1 id="PatrolID">0001</h1>'+
'<div id="bodyContent">'+
'<p><b>Nom de patrouille</b>, Coureur 1, Coureur 2, Coureur 3'+
'<br>Parti de Zermatt, Depart 22:00</p>'+
'</div></div>';

var infowindow = new google.maps.InfoWindow({
content: contentString});

//Function for the click on marker infowindow
google.maps.event.addListener(mark, 'click', function() {
infowindow.open(map, mark);});

//Path used by the patrol
var prevPath = \[
new google.maps.LatLng(46.03, 7.47),
new google.maps.LatLng(46.00, 7.40),
new google.maps.LatLng(45.99, 7.39),
new google.maps.LatLng(45.97, 7.37)\];

var PathPoly = new google.maps.Polyline({
path: prevPath,
strokeColor: "#00FF00",
strokeOpacity: 1.0,
strokeWeight: 3,
map : map});
}
//Initialization of the map on window load
google.maps.event.addDomListener(window, 'load', initialize);
</script>
</head>
<body>
<div id="map\_canvas" style="width: 1024px;height: 500px"></div>
</body>
</html>
```
This creates a basic map, initialises it with a static KML and adds a marker with a clickable information panel at a location set in JS.
