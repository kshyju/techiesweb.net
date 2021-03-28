---
layout: post
title: C# to Go - How to make HTTP call in Go
date: 2021-03-03 00:00:00.000000000 -05:00
tags: [CSharp to Go, C# to go, HttpClient in Go, REST call in Go, Http call in go]
description: C# developer's quick reference for Go snippets, How to make Http call in Go
---
## C# to Go - How to make HTTP call in Go

I have been dabbling with Go programming language lately(*really late to the party*😅) and surely liking this language. Me, being a C# developer, thought it might be a good idea to rewrite some of the C# snippets as part of learning Go. As I was doing this exercise, I decided to scribble it down here as a quick reference to help anyone taking the same route (C# developer learning Go) or for myself in the future.


Step 1, [Install go for your OS](https://golang.org/dl/).


I am going to skip the Hello world sample here for obvious reasons and will be doing the Go equivalent of some the the C# snippets we will use in real world application development.


## Http calls

C# Console program code for making Http call

    class Program
    {
        static async Task Main(string[] args)
        {
            Console.WriteLine("Http REST call demo");
            await MakeRestCallAsync("https://bing.com");
        }

        static async Task MakeRestCallAsync(string url)
        {
            try
            {
                using var httpClient = new HttpClient();
                using HttpResponseMessage response = await httpClient.GetAsync(url);
                response.EnsureSuccessStatusCode();
                Console.WriteLine($"{url} {(int)response.StatusCode} {response.StatusCode}");
            }
            catch (Exception ex)
            {
                Console.WriteLine($"{ex}");
            }
        }
    }

Go program (almost) equivalent for making Http call

    package main

    import (
        "fmt"
        "log"
        "net/http"
    )

    func main() {
        fmt.Println("Http REST call demo")
        makeRestCallAsync("https://bing.com")
    }

    func makeRestCallAsync(url string) {
        var resp, httpCallError = http.Get(url)
        if httpCallError == nil {
            fmt.Printf("%s %s", url, resp.Status)
        } else {
            log.Fatal(httpCallError)
        }
    }

You can check [this github repo](https://github.com/kshyju/CSharpToGo) for more samples. 

Cheers


