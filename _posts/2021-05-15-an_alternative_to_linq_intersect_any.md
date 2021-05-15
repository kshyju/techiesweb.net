---
layout: post
title: A fast alternative to Linq's Intersect.Any() expression result
date: 2021-05-15 00:00:00.000000000 -05:00
tags: [LINQ]
description: A fast alternative to Linq's Intersect.Any() expression result
---
## A fast alternative to Linq's Intersect.Any() expression result

While looking at the profiling data of one of our production systems, I stumpled across this intesteting block in the flamegraph. 


![linq taking 80 percent cpu](/assets/2021_5_15_before-linq-intersect.png)
 
Over 80 % of the time is spent in the linq expression where the `Any` extension method is called on the result of `Intersect` extension method.

Looking at the source code of the method, our goal of using this linq expression was to find there is a common element present between two string collections. Something like below.

<script src="https://gist.github.com/kshyju/d1903a06b84263de4a458f7046247dab.js?file=Blog2021LinqIntersectAnyUsageSample.cs"></script>

The profiler also tells us that this method took ~5 % of total CPU of our service process.

![total cpu usage of method](/assets/2021_5_15_before-total-cpu.png)

Looking at the source code of `Intersect` method, we can see it does two things.

 * Creates a Hashset and add items from first collection to it.
 * Tries to remove each items of second collection from this hash set. The Remove method returns `True` when it succesfully removes an item (a match has been found) and return those removed items(the common items).

![Intersect source code](/assets/2021_05_15_intersect-sourcecode.png)


The method returns an Enumerable collection(deferred execution). So If the `Any` method finds a match, it won't execute the code for remaining elements in second collection.

In my case, All I care about is finding out whether we have a common element present between these 2 collections. We could do this just with a nested for loop. So I ended up with this extension method.

<script src="https://gist.github.com/kshyju/d1903a06b84263de4a458f7046247dab.js?file=Blog2021MyIntersectAnyExtension.cs"></script>

This may look like a brute force solution. Would it do better compared to the LINQ expression? Only way to tell that by measuring both implementations. So I wrote benchmarking code for these 2 implementations.

<script src="https://gist.github.com/kshyju/d1903a06b84263de4a458f7046247dab.js?file=Blog2021MyIntersectAnyBenchmarks.cs"></script>


Benchmark results

<script src="https://gist.github.com/kshyju/d1903a06b84263de4a458f7046247dab.js?file=Blog2021MyIntersectAnyBenchmarkResults.md"></script>

And finally, after deploying the fix to prod, I went back and checked the prod profiler data again.

![total cpu usage of method](/assets/2021_05_15_after-getdata)
![total cpu usage of method](/assets/2021_05_15_after-total-cpu.png)

Cheers


