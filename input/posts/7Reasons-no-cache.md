Title: 7 reasons to not use caching
Published: 4/12/2022 16:30
Tags: [Thoughts, Caching, Redis, Architecture] 
---

Inspired by [Milan Jovanović](https://twitter.com/mjovanovictech) tweet on [5 reasons to use Redis for caching](https://twitter.com/mjovanovictech/status/1599124855542411264), 

![](/posts/images/caching/5reasonsCaching.png){width = 80%}

and [Daniel Marbach's](https://twitter.com/danielmarbach) response "[Now I want to see five reasons to avoid caching ✋😂](https://twitter.com/danielmarbach/status/1599352526888849408)"

![](/posts/images/caching/5reasonsNoCaching.png){width = 80%}

I found [seven reasons to not introduce caching](https://twitter.com/ThrowATwit/status/1599356806874427392): 

1. Caching can increase complexity in your application, as you need to manage the cached data and ensure it remains consistent with the underlying data store.

2. Caching can increase latency, as the cache itself introduces an additional lookup step.

3. Caching can be expensive, both in terms of the additional hardware and storage required for the cache, and the overhead of managing the cache itself.

4. Caching can be unreliable, as cached data can become stale or inconsistent if it is not adequately managed or invalidated.

5. Caching can be a security risk, as sensitive data that is stored in the cache may be vulnerable to unauthorized access or exposure. It takes additional effort to ensure that the correct authorizations are applied to cached data, increasing application complexity.

6. Caching can be harder to debug. To determine why a piece of data is not being retrieved from the cache or is being retrieved from the underlying data store instead is difficult. This can make it challenging to diagnose and fix performance issues related to caching.

7. Caching can create additional maintenance overhead, as you need to monitor the cache and ensure it is working properly. Monitoring cache hit and miss rates, ensuring that the cache is not getting too full, and periodically purging expired or stale data from the cache.

and a bonus [8.](https://mobile.twitter.com/joslat/status/1599518029649678336) from [Jose Luis Latorre](https://mobile.twitter.com/joslat)
"8. It should be also properly tested, and stress tested... without mention the security testing as well should include a check on this layer too... which would bring us to point 3. More expensive ;)"

Introducing Caching into any architecture is a decision that must be made with care. We have to ask if it helps us fulfill a business requirement (latency requirements), and improves quality or responsiveness for the end user. And we must ensure the solution is appropriate in terms of cost of operation and cost of monitoring and support. Additionally, the security aspects of a cache should be considered in the solution design.

In software architecture, there are very few single answers, everything is a compromise. Caching is a great hammer and use it when it is appropriate, but remember not every problem is a nail.