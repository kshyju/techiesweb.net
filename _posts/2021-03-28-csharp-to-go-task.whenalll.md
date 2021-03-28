---
layout: post
title: C# to Go - How to wait for an array of sub tasks to finish in Go
date: 2021-03-28 00:00:00.000000000 -05:00
tags: [CSharp to Go, C# to go, Task.WhenAll in Go, When.All in Go, Go Task.WhenAll, Go Wait for goroutines]
description: C# developer's quick reference for Go snippets, How to wait for an array of sub tasks to finish in Go
---
## C# to Go - How to wait for an array of sub tasks to finish

In C#, `Task.WhenAll` allows us to wait for a collection of tasks(which may be executed concurrently) to finish. In go, we can use `WaitGroup` from `sync` package along with goroutines to achieve similar thing.


## WhenAll

C# Console program code using `Task.WhenAll` to wait for the completion of a bunch of tasks running parallely.

    namespace ConsoleApp
    {
        using System;
        using System.Diagnostics;
        using System.Net.Http;
        using System.Threading.Tasks;

        class Program
        {
            static async Task Main(string[] args)
            {
                var start = Stopwatch.StartNew();

                var tasks = new Task[10];

                for (var i = 0; i < 10; i++)
                {
                    tasks[i] = MakeRestCallAsync("https://bing.com");
                }
                await Task.WhenAll(tasks);

                start.Stop();
                var elapsed = start.Elapsed;

                Console.WriteLine($"Total Elapsed time {elapsed.TotalMilliseconds}ms");
            }

            static async Task MakeRestCallAsync(string url)
            {
                try
                {
                    var start = Stopwatch.StartNew();

                    using var httpClient = new HttpClient();
                    using HttpResponseMessage resp = await httpClient.GetAsync(url);
                    resp.EnsureSuccessStatusCode();

                    start.Stop();
                    var elapsed = start.Elapsed;
                    Console.WriteLine($"{url} {(int)resp.StatusCode} {resp.StatusCode} Elapsed: {elapsed.TotalMilliseconds}ms");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"{ex}");
                }
            }
        }
    }


Go program (almost) equivalent waiting for parallel sub tasks.

    package main

    import (
        "fmt"
        "log"
        "net/http"
        "sync"
        "time"
    )

    func main() {

        var wg sync.WaitGroup

        start := time.Now()
        for counter := 0; counter < 10; counter++ {
            wg.Add(1) 
            // "go" keyword prefix to run the method as goroutine, achieving concurrency
            go makeRestCallAsync("https://bing.com", &wg)
        }

        wg.Wait()  // wait for all sub tasks to finish

        end := time.Now()
        elapsed := end.Sub(start)

        fmt.Printf("Total Elapsed time %s", elapsed)
    }

    func makeRestCallAsync(url string, wg *sync.WaitGroup) {
        start := time.Now()
        var resp, httpCallError = http.Get(url)
        if httpCallError == nil {
            end := time.Now()
            elapsed := end.Sub(start)
            fmt.Printf("%s %s Elapsed: %s \n", url, resp.Status, elapsed)

            wg.Done() // mark this instance of the task as done.
        } else {
            log.Fatal(httpCallError)
        }
    }


You can check [this github repo](https://github.com/kshyju/CSharpToGo) for more samples. 

Cheers


