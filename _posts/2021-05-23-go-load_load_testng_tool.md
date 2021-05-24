---
layout: post
title: Go-load, A simple load testing tool
date: 2021-05-23 00:00:00.000000000 -05:00
tags: [Go,GoLoad, Load testing tool, stress testing tool]
description: A simple load testing tool to generate load test traffic.
---
## Go-load, A simple load testing tool
As I was learning golang, I wrote a simple load testing tool in it. 

### Usage

#### Prerequisites
Make sure you have installed[ go runtime in your machine](https://golang.org/dl/)

#### Install go-load binary

Open a terminal/command prompt and run the below `go get` command which will install the go-load binary. This will download the source code to your machine and build the go-load.exe.

    go get github.com/kshyju/go-load

#### Command line options
* `-c` : Specifies the number of concurrent connections(goroutines) to use. You can consider this as the RPS you want for the test. Ex: `-c 50` will send 50 requests/second.
* `-d` : Specifies the duration of the load test in seconds. Ex: `-d 30` will run the tests for 30 seconds.
* `-h` : Specifies the request headers to send in a comma separated form. Each header item can use the `key:value` format. Ex: `my-apikey:foo,cookie:uid`
* `-body` : Specifies the path to a local file name which has the request payload data present. go-load will read the content of this file and use that as the request body. When this option is passed, Http POST method will be used to send the request.
* `-u` : Specifies the request URL explicitly.

## Examples

#### Send 10 requests/sec to bing.com for 30 seconds
    go-load -c 10 -d 30 https://www.bing.com

If you have special characters like `&` in your URL, you need to pass the value in quotes.

    go-load -c 10 -d 30 "https://www.bing.com?how=are&you=today"

#### Sending request headers
Request headers can be passed as a comma separated string with the `-h` flag. The string should have header key and value in the `key:value` format. Ex: `User-Agent:Go-http-client/1.1` 

    go-load -c=10 -d=30 -h my-apikey:foo,cookie:uid:bar https://www.bing.com

The above example is sending 2 request headers, "my-apikey" and "cookie".

#### Sending HTTP POST request with payload from a local file

    go-load -c=10 -d=30 -body="C:\\temp\\my-payload.json" "http://your.app/which/accepts/http-post?foo=bar"

At the end of the run, go-load will generate a run summary which includes latency(different percentiles) and aggregated status code count.

![Go-Load run summary](/assets/2021_05_23_go-load-result-summary.png)

[Source code](https://github.com/kshyju/go-load?from=tw)

Cheers


