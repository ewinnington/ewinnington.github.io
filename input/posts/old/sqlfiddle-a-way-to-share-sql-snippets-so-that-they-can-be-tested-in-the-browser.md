Title: SqlFiddle - A way to share SQL snippets so that they can be tested in the browser
Published: 05/11/2013
Tags: [Migrated, SQL, Database] 
---

A couple of days ago, I stumbled upon [SQLFiddle](http://www.sqlfiddle.com/), a little magical website that allows you to upload a DBSchema and run queries on it, live in the browser. It also allows to share them with other users.

In the course of my work, I've often come across a table that has an identifier, a timestamp and a value, and these rows must be mapped to one new row containing multiple identifiers. Two typical examples of that are two identifiers defining a minimum and a maximum value timeseries that must be put into a single row entry and an entry that is built from three identified timeseries representing value, volatility and mean reversion parameters of a stochastic process.

In my earlier blog post, [Oracle 10g - Pivoting data](Oracle10g-Pivoting-data.html), I provided two different sql scripts to solve the first case min/max time series. You can find the SQLFiddles here:

[Decode with subquery and max ](http://www.sqlfiddle.com/#!4/dce2d/1)

[Decode in one query and max ](http://www.sqlfiddle.com/#!4/dce2d/2)

You can even view your query's execution plan, which is a really nice touch. I'll take the time to share some more oracle sql statements next time in an SQLFiddle.
