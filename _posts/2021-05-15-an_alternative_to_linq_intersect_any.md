---
layout: post
title: A fast alternative to Linq's Intersect.Any() expression result
date: 2021-05-15 00:00:00.000000000 -05:00
tags: [LINQ]
description: A fast alternative to Linq's Intersect.Any() expression result
---
## A fast alternative to Linq's Intersect.Any() expression result

While looking at the profiling data of one of our production systems, I stumbled across this interesting block in the CPU flame graph. 


![linq taking 80 percent cpu](/assets/2021_5_15_before-linq-intersect.png)
 
Let's call the obscured method `GetData`. Over 80 % of the time in `GetData` is spent in the LINQ expression where the `Any` extension method is called on the result of `Intersect` extension method. This `GetData` method was obviously in hot path of the application, getting executed multiple times per incoming request.

The goal of using this LINQ expression in our `GetData` method was to find whether there was a common element present between two string collections. Something like below.

<script src="https://gist.github.com/kshyju/d1903a06b84263de4a458f7046247dab.js?file=Blog2021LinqIntersectAnyUsageSample.cs"></script>

The profiler also tells us that this method took ~5 % of total CPU of our service process.

![total cpu usage of method](/assets/2021_5_15_before-total-cpu.png)

Looking at the source code of `Intersect` method, we can see it does two things.

 * Creates a Hash set and add items from first collection to it.
 * Tries to remove each items of second collection from this hash set as we iterate through the result of this method. The Remove method returns `True` when it successfully removes an item (a match has been found) and return those removed items(the common items).

![Intersect source code](/assets/2021_05_15_intersect-sourcecode.png)


The `Intersect` method returns an Enumerable collection(deferred execution). So If the `Any` method returns `True`, it won't execute the code for remaining elements in second collection.

In my case, All I care about is finding out whether we have a common element present between these 2 collections. We could do this just with a nested for loop without allocating the HashSet. We could also eliminate the hash generation during the Remove method call. So I ended up with this extension method.

<script src="https://gist.github.com/kshyju/d1903a06b84263de4a458f7046247dab.js?file=Blog2021MyIntersectAnyExtension.cs"></script>

This may look like a brute force solution. Would it do better compared to the LINQ expression? Only way to tell that by measuring both implementations. So I wrote benchmarking code for these 2 implementations.

<script src="https://gist.github.com/kshyju/d1903a06b84263de4a458f7046247dab.js?file=Blog2021MyIntersectAnyBenchmarks.cs"></script>


The benchmark results

<script src="https://gist.github.com/kshyju/d1903a06b84263de4a458f7046247dab.js?file=Blog2021MyIntersectAnyBenchmarkResults.md"></script>

The benchmark results clearly shows:
 * When both the source collections are Arrays, the custom implementation is 7 times faster than the LINQ expression. Also it allocates 0 bytes (no hash code allocation in managed heap. The for loop runs in the stack)
  * When both the source collections are Lists, the custom implementation is 4.5 times faster than the LINQ expression. There is allocation where our `AsArray` method has to convert the collection to an Array. But still a lot smaller than what the LINQ expression allocated.

And finally, after deploying the fix to prod, I went back and checked the prod profiler data again.

![total cpu usage of method](/assets/2021_05_15_after-getdata.png)
![total cpu usage of method](/assets/2021_05_15_after-total-cpu.png)

The `GetData` method now consumes 1.6 % CPU where it used to consume around 5 %.

Important take away is, 
 * LINQ expression is concise, but may not be optimal if that is being used in a hot path.
 * Measure, Fix, Measure again, Repeat.

Cheers


