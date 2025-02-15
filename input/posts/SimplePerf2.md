Title: In-memory software design 2025 - Applied to energy
Published: 15/02/2025 08:00
Tags: [Architecture, Cpp] 
---

# In-memory software design 

In the last blog, [In-memory performance in 2025](https://ewinnington.github.io/posts/SimplePerf) we looked at a simple design for an energy aggregation system to aggregate 9.8 GB of 100'000 trades rolled out and saw we got about 40 GB/s performance. 

I upped the numbers of trades to 500'000 trades (rolled out over quarter hours, with average trade length of 120 days in quarter hours) and validated that on a ~50 GB dataset, we are running at ~1200 ms for the complete aggregation, confirming a linear scaling of the compute time. 

But this brings us to a point where on commodity hardware, we are running our aggregations over 1 second in duration. So how can we improve this? This time, instead of trying to brute force, let's bring in some other techniques from databases: Indexes!

## Simple indexes for in-memory aggregations

For our use case, we are going to look at two meta-data properties on our trades, Trader and Delivery area, but the concept scales efficiently to large collections of meta-data, because we are dealing with such a small amounts of Trades (500k).

### Building the index

We can generate at insertion a set for each meta-data, trader and delivery area. 

```cpp
    // =================
    // 2) Build Indexes
    // =================
    // We'll do single-attribute indexes: trader -> set of tradeIDs, area -> set of tradeIDs
    // For large data, consider using std::vector<size_t> sorted, or some other structure.

    std::unordered_map<std::string, std::unordered_set<size_t>> traderIndex;
    traderIndex.reserve(10);  // if you know approx how many traders you have to avoid resizing continuously

    std::unordered_map<std::string, std::unordered_set<size_t>> areaIndex;
    areaIndex.reserve(10);
```

I'm using an unordered set, but could also use an ordered set, this might be more efficient. 

As keys, I'm using the string data of the meta-data on the trade - in production this would probably be integer keys, but for this use case is sufficient. 

```cpp
    for (size_t i = 0; i < trades.size(); ++i)
    {
        const auto& t = trades[i];
        traderIndex[t.trader].trades(i);
        areaIndex[t.deliveryArea].trades(i);
    }
```

When we insert a set of trades, we can add them to the set index. 

As a linear pass, it's extremely efficient to build this index and cheap to keep it updated as we insert new trades.

### Using the index 

If we have a search query on a single attribute, we can now use the simple index and have directly the result. 

But if we are doing a query on two (or more), we are going to take use the smallest index first and match with the largest index. The unordered_set gives an acceptable performance for `bigger.find(id)` but you can probably do even better with a set structure that is optimized for intersections. You can benchmark using `std::set_intersection` against my simple implementation if you are using sorted sets. 

```cpp
// ===================================
    // 3) Use the indexes for filtering
    // ===================================
    // For instance, let's do a query: TraderX, Area2
    // We'll find the intersection of (all trades for TraderX) and (all trades for Area2).

    std::string queryTrader = "TraderX";
    std::string queryArea   = "Area2";

    // Get sets from the index
    // (handle the case if the key doesn't exist -> empty set)
    auto itT = traderIndex.find(queryTrader);
    auto itA = areaIndex.find(queryArea);

    if (itT == traderIndex.end() || itA == areaIndex.end()) {
        std::cout << "No trades found for " << queryTrader << " AND " << queryArea << "\n";
        return 0;
    }

    const auto& traderSet = itT->second;
    const auto& areaSet   = itA->second;

    // Intersection
    // We'll create a vector of trade IDs that are in both sets
    // For speed, we can iterate over the smaller set and check membership in the larger set.
    const auto& smaller = (traderSet.size() < areaSet.size()) ? traderSet : areaSet;
    const auto& bigger  = (traderSet.size() < areaSet.size()) ? areaSet   : traderSet;

    std::vector<size_t> intersection;
    intersection.reserve(smaller.size());  // a safe upper bound if all the smaller set is selected, to avoid resizing

    for (auto id : smaller)
    {
        if (bigger.find(id) != bigger.end())
        {
            intersection.push_back(id);
        }
    }
```

This gives us the intersection and we iterate it to do the summation - again, here we are summing up over the entire year into a single value, but we could just as easily be doing daily, weekly or monthly sums. 

```cpp
    // ====================================
    // 4) Aggregate deliveries for matches
    // ====================================
    // Let's sum up total power / total value for the intersection set.

    long long totalPower = 0;
    long long totalValue = 0;

    #ifdef _OPENMP
    #pragma omp parallel for num_threads(4) reduction(+:totalPower, totalValue)
    #endif
    for (auto id : intersection)
    {
        const Trade& t = trades[id];
        // sum up all deliveries
        for (const auto& dd : t.dailyDeliveries)
        {
            for (int slot = 0; slot < 100; ++slot)
            {
                totalPower += dd.power[slot];
                totalValue += dd.value[slot];
            }
        }
    }
```

This addition here is single threaded, but you can also use OpenMP to accelerate it - this will require some tuning, you don't want to use too many threads for these smaller aggregations, the `omp parallel for num_threads(4)` can be added to example limit to 4 threads. (note: I added the openMP in the code above). 

Generally you get a 10x or more acceleration in single core, depending on how selective the indexes you are using are- In parallel, I'm getting 40-50x acceleration in multi-core with a num_threads to 4. 

## Why array<int,100> ? 

In my last post, I used an array of 100 points in a day. I'm using it because it's simplest to have all days have the same "size" for memory alignment, therefore if I am calculating days in local time I have a 25h and 23h hour day once a year. I would generally prefer to work in UTC and have constant 24h days -- but for some reason humans prefer local time so it aligns with expectations.

Just be clear with the developer if you use an internal UTC or Local time representation. If using local make sure to: 

- Properly sanitize your inputs, check trades fill the 96 quarter hours only for all days apart from the short (92) and long day (100) and zero fill the remainder. 
- Keep summing on all 100 hours for summations, the 4% extra index length is not worth an if statement in the inner loop of the code - try to keep loops jump free.

## How to deal with Canceled / Amended / Recalled trades ? 

My first answer is don't! Let me explain: Normal trades should represent the 99.9% or 99.99% of your deals - unless there's something you haven't told me about the way you are trading!

We can design this by having a tradeStatus on the trade.

int TradeStatus:
- 0 : trade is valid
- 1 : trade is canceled 
- 2 : trade is amended (ie. replaced by a new one)
- ... : any other status necessary 

When a trade is canceled, we leave it in the trade vector, but simply set the tradeStatus to a non-zero value, and skip it with a test at the beginning of the aggregation. 

```cpp
    #ifdef _OPENMP
    #pragma omp parallel for num_threads(4) reduction(+:totalPower, totalValue)
    #endif
    for (auto id : intersection)
    {
        const Trade& t = trades[id];
        if(t.tradeStatus != 0) continue;  // skip canceled/amended trades
```

If the trade is amended, same thing, we add a new trade to our list of trades and set the previous one to amended status. Generally, the this is not using up much memory. If it ever becomes a problem, we could: 

a) have a "trade compression" which removes all non-zero trade status from the vector. 
b) flush the entire trade vector and reload the whole set. 

Depending on your implementation, the flushing and reloading might be just as fast - not every program needs to stay resident in memory all the time. 

## Snapshot state to disk for recovery or storage - Fork() as in Redis

If we want to take snapshots of the state,  we can get inspired from Redis' famous [fork() snapshotting technique](https://architecturenotes.co/i/143231289/forking). 

Use this when needing to snapshot a large data structure in RAM to disk (serialize the entire state), without blocking our main process from accepting new trades for the entire duration of the write.

### How fork() helps

On Linux, calling fork() creates a child process that initially shares the same physical memory pages as the parent.

Copy-on-write (CoW): If either the parent or the child writes to a page after the fork, the kernel duplicates that page so each process sees consistent data.

The child process can serialize the in-memory data (in a consistent state from the moment of forking) to disk, while the parent continues to run to accept new trades. New trades arriving in the parent process after the fork will not affect the child’s view of memory. The child effectively sees a snapshot as of the fork().

You want the data structure to be in a consistent state at the instant of fork().
A brief lock (or pause writes) just before the fork() is triggered, ensuring no partial updates. Immediately after fork() returns, you can unlock, letting the parent continue. Meanwhile, the child proceeds to write out the data.

We can store an atomic counter value in the program that represents the last tradeid inserted or a state version. This gives you a “version” or “stamp” number for the dataset.

I won't put the full code for that here, since the design is a little more involved, but the basics are: 

```
static std::atomic<long> g_version{0}; //snapshot version id 
static std::mutex g_tradeMutex;  // protect g_trades from concurrent modification - lock on write to 

// ---------------------------------------------------
// fork() to create a child that writes the snapshot
// ---------------------------------------------------
int snapshotNow(const char* filename) {
    // 1) Acquire short lock to ensure no partial updates in progress
    g_tradeMutex.lock();
    long snapVer = g_version.load(std::memory_order_relaxed);

    // 2) Fork
    pid_t pid = fork();
    if(pid < 0) {
        // error
        std::cerr << "fork() failed\n";
        g_tradeMutex.unlock();
        return -1;
    }

    if(pid == 0) {
        // child
        // We have a consistent view of memory as of the fork.
        // release the lock in the child
        g_tradeMutex.unlock();

        // write the snapshot
        writeSnapshotToDisk(filename, snapVer);

        // exit child
        _exit(0);
    } else {
        // parent
        // release the lock and continue
        g_tradeMutex.unlock();
        std::cout << "[Parent] Snapshot child pid=" << pid 
                  << ", version=" << snapVer << "\n";
        return 0;
    }
}
```

In essence we have two processes continuing from the same command fork() return, each taking one branch. 

## Data I/O 

To integrate your cpp aggregation software into the rest of your stack depends on the software running around it. 

You can run the application as an on-demand aggregation, loading everything to memory, doing the aggregation and exiting - leaving the server to do something else - this can be worth it if you only do aggregations on-demand and can afford the load time of a second or two from your NVME storage.

You can keep the cpp program running either : 
- exposing an http RestAPI (https://github.com/microsoft/cpprestsdk is a good library for that, I've used it before).
- having a GRPC endpoint for performance
- receiving data from a Kafka stream - I'm sure Confluent can give you a good example of that. 

## Summary

In-memory data aggregation using cpp is relatively easy to write and maintain. Cpp is no longer the terrible monster it was - auto pointers help and using std:: components makes everything simple. OpenMP is an easy win to add to compute or memory intensive sections.



#### Sidenotes 

On Windows you can get everything you need to compile cpp by installing: 

```
winget install LLVM.LLVM
```

On Linux (or wsl), you need to install the following: 
```
sudo apt-get update
sudo apt-get install clang
sudo apt-get install libomp-dev 
```

Then in both os, you can usually run a compilation on your source file (inmem\_agg\_omp.cpp) using: 

```
clang++ -O3 -march=native -flto -ffast-math -fopenmp -o inmem_agg_omp inmem_agg_omp.cpp
```
