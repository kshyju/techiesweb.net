---
layout: post
title: C# to Go - How to use StopWatch in Go
date: 2021-03-21 00:00:00.000000000 -05:00
tags: [CSharp to Go, C# to go, Stopwatch in Go, Measure method execution time in Go, Go stopwatch]
description: C# developer's quick reference for Go snippets, How to use StopWatch in Go
---
## C# to Go - How to measure elapsed time

StopWatch type in C# provides an easy way to measure the time it took for a specific method execution. In go, we can use the `time` package to do similar thing.


## Stopwatch

C# Console program code using StopWatch to record elapsed time.

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

                for (var i = 0; i < 5; i++)
                {
                    await MakeRestCallAsync("https://bing.com");
                }

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
                    Console.WriteLine($"{url} {(int)resp.StatusCode} {resp.StatusCode} Elapsed time: {elapsed.TotalMilliseconds}ms");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"{ex}");
                }
            }
        }
    }


Go program (almost) equivalent for making Http call

    package main

    import (
        "fmt"
        "log"
        "net/http"
        "time"
    )

    func main() {

        start := time.Now()
        for i := 0; i < 5; i++ {
            makeRestCallAsync("https://bing.com")
        }
        end := time.Now()
        elapsed := end.Sub(start)

        fmt.Printf("Total Elapsed time %s", elapsed)
    }

    func makeRestCallAsync(url string) {
        start := time.Now()
        var resp, httpCallError = http.Get(url)
        if httpCallError == nil {
            end := time.Now()
            elapsed := end.Sub(start)

            fmt.Printf("%s %s Elapsed time: %s \n", url, resp.Status, elapsed)
        } else {
            log.Fatal(httpCallError)
        }
    }

You can check [this github repo](https://github.com/kshyju/CSharpToGo) for more samples. 

Cheers


