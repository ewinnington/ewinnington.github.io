Title: R - Reading JSON from redis
Published: 3/11/2019
Tags: [R, JSON, Redis] 
---

I needed to access Redis from R. Here's the code I used to connect, set a value, then deserialize a key that was in json format and publish a message. [Rredis documentation located here](https://cran.r-project.org/web/packages/rredis/vignettes/rredis.pdf)

```
library("rredis")
library("jsonlite", lib.loc="~/R/win-library/3.2")

redisConnect()
redisSet("x",rnorm(5))
redisGet("x")

x = fromJSON(redisGet("ts_1"))
x$t <-  strptime(x$t, "%Y-%m-%dT%H:%M:%S")

//https://cran.r-project.org/web/packages/rredis/vignettes/rredis.pdf
redisPublish("General:Active", charToRaw('Message'))

redisClose()
```
