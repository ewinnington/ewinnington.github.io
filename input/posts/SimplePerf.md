Title: Performance of in-memory in 2025 
Published: 13/02/2025 08:00
Tags: [Architecture, Cpp] 
---

# Performance of in-memory software in 2025

Yesterday, at an Energy panel in Essen, I mentioned that some heavy calculations should be done in-memory. It is something that people have a tendency to dismiss because generally they are not aware of the capability and speed of modern CPUs and RAM. A 32GB dataset in memory can now be processed every second by a commodity cpu in your laptop. 

[Living in the Future, by the numbers](https://tailscale.com/blog/living-in-the-future) is a great article on the progress we have had since 2004: 
- CPU Compute is 1000x faster
- Web servers are 100x faster
- Ram is 16x to 750x larger
- SSD can do 10'000x more transactions per second.

You can also see this progression on [Azure with the high-compute servers](https://techcommunity.microsoft.com/blog/azurehighperformancecomputingblog/announcing-azure-hbv5-virtual-machines-a-breakthrough-in-memory-bandwidth-for-hp/4303504), Microsoft and AMD are packing so much more memory bandwidth in modern compute. 

![image](https://github.com/user-attachments/assets/b7561026-767e-4dc4-97a5-e5306c3fa36a){ width=60% }

We are going to have 7 TB/s of memory bandwidth!

![image](https://github.com/user-attachments/assets/f2c78f96-0eb0-4359-9f65-963b2b4b4f7b){ width=60% }

You can now run super-computer level problems on Azure!

But what does that all mean? What can we do even on a commodity laptop? I have a Latitude 9440 laptop on my desk here, with a 13th gen i7-1365U with 32 GB of RAM. 

## Energy Trade aggregation 

Let's start with a small calculation from the world of Energy. I have 100'000 trades, these trades affect one or multiple quarter hours of one year (8'784 hours => 35'136 quarter hours). 

Pulling out a little C++, completely unoptimized, how long does it take to aggregate them and how much RAM is used? 


```cpp
// A struct to hold the daily delivery arrays (power, value).
struct DailyDelivery
{
    int dayOfYear;  // 1..365
    std::array<int, 100> power; 
    std::array<int, 100> value; 
};

// A struct to hold the trade metadata.
struct Trade
{
    int tradeId;
    std::string trader;        // e.g. "TraderX"
    std::string deliveryArea;  // e.g. "Area1", "Area2"
    // You could store time points, but we'll just store day indexes for simplicity.
    // Real code might store start_delivery, end_delivery as std::chrono::system_clock::time_point.
    int startDay; // 1..365
    int endDay;   // 1..365

    std::vector<DailyDelivery> dailyDeliveries;
};
```
The Daily Delivery structure represents a day of delivery, with 100 slots (for the 25h day => 100 quarter hours). 
- I'm storing the delivery of power in kW as an int32, meaning in a single trade I can do –2'147'483'648 kW to 2'147'483'647 kW.  
- Same thing for the value, we store the individual value of the MW in milli values (decimal shift 3), so each MW could be priced at -2'147'483.648 € to 2'147'483.647 €. 

The Trade stores: 
- metadata: Trader and DeliveryArea. We could add as many metadata elements as we need, but for simplicity in the demo, I only use this
- a dailyDeliveries vector containing the array of all days affected by the trade.

Now if we wanted to see what is the total sum of power of all trades, the sum of TraderX and the sum of TraderX's deals in Area1 and Area2, we can runn the aggregation over all the memory. This is completely straight forward code, no optimizations what so ever. 

```cpp
    // --------------------------------------
    // 2) Run the aggregations (measure time)
    // --------------------------------------

    // The aggregates we want:
    // (a) All Trader total (yearly - all zones)
    // (b) Trader X total
    // (c) Trader X / Area1
    // (d) Trader X / Area2
    //
    // We'll assume we only have TraderX, so "All Trader" == "TraderX" in this simple version.
    //
    // But let's keep it generic. If you had multiple traders, you'd do some checks:
    //
    // For Weighted Average Cost = total_value / total_power (where total_power != 0)

    // We'll measure the time for a single pass that gathers all these sums.

    using Clock = std::chrono::steady_clock;
    auto startTime = Clock::now();

    long long all_totalPower = 0;
    long long all_totalValue = 0;

    long long traderX_totalPower = 0;
    long long traderX_totalValue = 0;

    long long traderX_area1_totalPower = 0;
    long long traderX_area1_totalValue = 0;

    long long traderX_area2_totalPower = 0;
    long long traderX_area2_totalValue = 0;

    for(const auto& trade : trades)
    {
        // (a) "All Trader" sums:
        //    Summation for all trades, all areas, all days
        //    Because this example is all TraderX, you might have to adapt if you had multiple traders
        for(const auto& dd : trade.dailyDeliveries)
        {
            for(int slot = 0; slot < 100; ++slot)
            {
                all_totalPower += dd.power[slot];
                all_totalValue += dd.value[slot];
            }
        }

        // (b) If trade.trader == "TraderX"
        if(trade.trader == "TraderX")
        {
            for(const auto& dd : trade.dailyDeliveries)
            {
                for(int slot = 0; slot < 100; ++slot)
                {
                    traderX_totalPower += dd.power[slot];
                    traderX_totalValue += dd.value[slot];
                }
            }

            // (c) and (d) by area
            if(trade.deliveryArea == "Area1")
            {
                for(const auto& dd : trade.dailyDeliveries)
                {
                    for(int slot = 0; slot < 100; ++slot)
                    {
                        traderX_area1_totalPower += dd.power[slot];
                        traderX_area1_totalValue += dd.value[slot];
                    }
                }
            }
            else if(trade.deliveryArea == "Area2")
            {
                for(const auto& dd : trade.dailyDeliveries)
                {
                    for(int slot = 0; slot < 100; ++slot)
                    {
                        traderX_area2_totalPower += dd.power[slot];
                        traderX_area2_totalValue += dd.value[slot];
                    }
                }
            }
        }
    }

    auto endTime = Clock::now();
    auto durationMs = std::chrono::duration_cast<std::chrono::milliseconds>(endTime - startTime).count();
```

How long do you think that takes on a commodity laptop? It's 9.8 GB of RAM to scan and fully aggregate. This is also running inside a VM on my Windows WSL instance, with other software running at the same time. 

```
Time for in-memory aggregation: 907 ms
--- Memory usage statistics (approx) ---
Total bytes allocated (cumulative): 9822202136 bytes
Peak bytes allocated (concurrent):  9822202136 bytes
Current bytes allocated:            9822202136 bytes
```

## Parallelization

Since we are running on a multicore CPU, we can use more than one core to do the aggregation. With OpenMP, it's extremely simple to setup some parallelization for the compute. At the beginning of the loop, we can define a parallel aggregation for reduction, meaning a final sum.
```cpp 
    // Parallel over trades
    // The 'reduction(+: variableList)' tells OpenMP to create private copies of
    // these variables in each thread, accumulate them, and then combine them
    // at the end.
#ifdef _OPENMP
#pragma omp parallel for reduction(+ : all_totalPower, all_totalValue, \
                                       traderX_totalPower, traderX_totalValue, \
                                       traderX_area1_totalPower, traderX_area1_totalValue, \
                                       traderX_area2_totalPower, traderX_area2_totalValue)
#endif
    for (std::size_t i = 0; i < trades.size(); ++i)
    {
        const auto& trade = trades[i];
```

With this, we improve the time to aggregate on the laptop to: 241ms. This means we can now do the **complete aggregation on a laptop 4x per second** - even on a completely unoptimized, simplistic memory structure for trades. 
```
Time for in-memory aggregation: 241 ms
```

So when you are doing large numerical aggregations or calculations, ask yourself - can I do this in RAM? If so, you might be surprised at how quickly and efficiently you can do it with modern cpp.

## Performance vs Memory bandwidth 
This completely unoptimized implementation is running at : 

Data size (GB) / time (s) = bandwidth (GB/s) => 9.8 GB / 0.241 s ≈ **40.7 GB/s**  

My laptop has approx \~96 GB/s of memory bandwidth. I calculate it as follows: LPDDR5 at 6000 MT/s, 8 bytes, dual channel = 6000 * 8 * 2 =~ 96 GB/s. My laptop has less than half bandwidth of the HC family on Azure (AMD EPYC™ 7003-series CPU) using CPUs that were released in 2021. Still impressive for my laptop, but it shows you could do much better. 

If I really needed to optimize, I would re-organise the data structures to improve the memory aligment as an initial step. With that, we should get closer to the theoretical bandwidth of the machine. 
