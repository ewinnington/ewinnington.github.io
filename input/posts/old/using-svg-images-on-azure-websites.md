Title: Using SVG images on Azure Websites
Published: 23/08/2014
Tags: [Migrated, SVG, Azure, Websites] 
---


I've been using [Iconic's icon set](https://useiconic.com/) for the development of a dashboard for one of my projects. While publishing it to [Azure](http://azure.microsoft.com/en-us/) to test it out, none of the SVGs images or icons were being served.

It required a little bit of configuration to get it all running:

1) Make sure the svg files are included in the visual studio project and are set to "Copy Always" or "Copy If Newer".

2) Configure the web.config of your project to contain the following code to make azure serve the SVGs.
```
 <system.webServer>
   <staticContent>
     <remove fileExtension=".svg"/>
     <mimeMap fileExtension=".svg" mimeType="image/svg+xml" />
   </staticContent>
 </system.webServer>
```
This give the correct mimeType to Azure. For some strange reason, it doesn't have it by default in its configuration, and you'll get errors when trying to serve the content (404). With these steps done, Azure should be able to serve you static SVG images when you use the correct path (I use /img/ ).
```
<img src="~/img/database.svg" alt="database">
```
This should get you a black and white database icon.

3)  To use Iconic's features like CSS styling of the icons, you need to include in your script bundle the iconic.js (or iconic.min.js).

4) Use the data-src attribute in the <img> tag to indicate the Iconic that it must inject the SVG into the document.
```
<img data-src="~/img/database.svg" class="iconic iconic-md ok" alt="database">
```
5) Create custom CSS markings like the "ok" class to style the icon. I added these in a site-iconic.css file that I include in the bundles.

.ok {
 fill: green;
}
.nok {
 fill: red;
}

This "ok" class changes the icon fill to green, I have similarly an "nok" class changing the icon to red.

6) If you are using the Razor engine to generate your pages, you can directly assign class names based on the object's properties. Here I have an object DbServer which has a boolean property isAvailable to show its status.
```
@{
string status = (DbServer.isAvailable ? "ok" : "nok");
<img data-src="~/img/database.svg" class="iconic iconic-md @status" alt="database">
}
```
Now, every time the page is refreshed, the status of the DB is indicated by the colour of the Database icon.

[![Dashboard](old/images/Dashboard-e1408819237573.png)](old/images/Dashboard-e1408819237573.png)

Now you can really use Iconic's icon set potential.

Of course, if you are using [SignalR](http://signalr.net/), you can also wire up the whole setup to dynamically adjust the icons with javascript, but that's for another post.
