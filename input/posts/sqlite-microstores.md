Title: Embracing SQLite and living with micro-services
Published: 22/10/2022
Tags: [SQLite] 
---

The idea of micro-services and their own single purpose data stores is easy to describe. But then to implement and live with it is a different story. So as a developer and architect, I’ve decided to do just that! Make micro-services and micro-data stores to cover the tiny and small stuff in my life I want to keep track of. 

As an example, I read online comics, light novels and mangas. I had a continuous list of a couple hundred bookmarks that I tried to keep updated with the last position I was when I read the story. But I always forget to update the bookmark and have so many of them that I lose the last read chapter. My solution? 

A SQLite db and some Python code to load it. Pass a single url on some command line Python and it gets added to the Db, a Request goes out, gets the title and chapter from the html, then adds it to the DB by title. Now I have a track of where I left off and I can get have last updated / last read records. Bonus, I can do a SELECT .. ORDER BY updated LIMIT 10 to check the last stories I was reading and pipe them to my browser to open up the chapters where I left them off.

To really embrace SQLite is to make everything in your life become a new micro database, even if there's only a couple of tables with a dozen or a hundred rows.

Stock tracking? an SQLite Db with Transactions and a roll-up Inventory table.

In fact, even when sending data around from one system to another, we should even embrace the simplicity of SQLite over CSV files. See https://berthub.eu/articles/posts/big-data-storage/ for his views and performance tests.

Now I have micro data-stores, I can add a service on top which contains the CRUD commands I need to interact with them and show them in a personal dashboard.