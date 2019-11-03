Title: KML File structure
Published: 24/04/2012
Tags: [Migrated, KML] 
---

I needed to make a KML suitable for reading in Google Earth. Here's the template:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<kml xmlns="http://www.opengis.net/kml/2.2">
<NetworkLinkControl> <!-- Set minimum refresh to 3 minutes -->
 <minRefreshPeriod>180</minRefreshPeriod> 
</NetworkLinkControl>
<Document>
<name>pdg2012.kml</name>
<!-- Definition des styles du KML -->
<Style id="My\_Style1">
 <IconStyle> <Icon> <href>ball.png</href> </Icon></IconStyle> <!-- Transparent PNG must be in same folder on server -->
</Style> 
<Style id="My\_Style2">
 <IconStyle> <Icon> <href>ball2.png</href> </Icon></IconStyle>
</Style> 
<Folder>
<name>Patrouille des glaciers</name>
<Placemark><name>Dossard 1</name>
<description>Echo, Foxtrot, Golf</description>
<Point><coordinates>7.35162,46.22549,0</coordinates></Point>
<styleUrl>#My\_Style1</styleUrl> 
</Placemark>
<Placemark><name>Dossard 2</name>
<description>Alpha, Bravo, Charlie</description>
<Point><coordinates>7.35196,46.2254,0</coordinates></Point>
<styleUrl>#My\_Style2</styleUrl> <!-- Styles must be defined-->
</Placemark>
</Folder>
</Document>
</kml>
```
The minRefreshPeriod indicates to Google Earth not to load the KML file too often when a network link is directed to it.
