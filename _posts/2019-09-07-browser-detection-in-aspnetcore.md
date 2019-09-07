---
layout: post
title: Browser detection in ASP.NET Core apps
date: 2019-09-07 00:00:00.000000000 -05:00
tags: [BrowserDetection, Browser detection, DeviceTypeDetection, AspNet core Browser detection, Operating System detection]
description: Browser detection in asp.net core and device detection in asp.net core

---

.NET framework 4.7 has a `Browser` property on `HttpContext.Request` which gives you information about the browser, from where the HTTP request came from. Unfortunately, ASP.NET core does not have this feature and the ASP.NET team decided to not port it.

Here is the [response from the team](https://github.com/aspnet/AspNetCore/issues/7033).

> Hi, the browser caps feature in ASP NET 4.x was a server-based way of doing client feature detection. This approach is generally frowned upon in the "modern" era, where runtime client-side feature detection is preferred, using techniques such as progressive enhancement. For that reason, in addition to being a huge maintenance cost to keep the list even reasonably up-to-date, the feature was not brought forward to ASP NET Core.

While I agree with the above response, IMHO, There are certain use cases where browser detection needs to be done on server side. A good example is a proxy server acting as a gateway for requests coming to your web server. Based on the browser, the gateway can route the traffic to different versions of your app/servers (think about an A/B testing infrastructure, targeting a user base from a specific browser / browser version).

I ended up writing a light weight browser detection library.

This library adds the following capabilities to your asp net core app

1. Browser detection
2. Device type detection
3. Operating system detection

## How to use ?

**Step 1:**
Install the [BrowserDetector nuget package](https://www.nuget.org/packages/Shyjus.BrowserDetector/)


{% highlight c# %}
Install-Package Shyjus.BrowserDetector
{% endhighlight %}

**Step 2:** Enable the browser detection service inside the `ConfigureServices` method of `Startup.cs`.

{% highlight c# %}
public void ConfigureServices(IServiceCollection services)
{
    // Add browser detection service
    services.AddBrowserDetection();

    services.AddMvc();
}

{% endhighlight %}

**Step 3:** Inject `IBrowserDetector` to your controller class or view file or middleware and access the `Browser` property. The property is lazy loaded.

Example usage in controller code

{% highlight c# %}
public class HomeController : Controller
{
    private readonly IBrowserDetector browserDetector;
    public HomeController(IBrowserDetector browserDetector)
    {
        this.browserDetector = browserDetector;
    }
    public IActionResult Index()
    {
        var browser = this.browserDetector.Browser;
        // Use browser object as needed.

        return View();
    }
}
{% endhighlight %}

Example usage in view code

{% highlight c# %}
@inject Shyjus.BrowserDetector.IBrowserDetector browserDetector

<h2> @browserDetector.Browser.Name </h2>
<h3> @browserDetector.Browser.Version </h3>
<h3> @browserDetector.Browser.OS </h3>
<h3> @browserDetector.Browser.DeviceType </h3>

{% endhighlight %}

Example usage in custom middlware

You can inject the `IBrowserDetector` to the `InvokeAsync` method.

{% highlight c# %}
public class MyCustomMiddleware
{
    private RequestDelegate next;

    public MyCustomMiddleware(RequestDelegate next)
    {
        this.next = next;
    }

    public async Task InvokeAsync(HttpContext httpContext,
                                  IBrowserDetector detector)
    {
        var browser = detector.Browser;

        if (browser.Name == BrowserNames.Edge)
        {
            await httpContext.Response
                  .WriteAsync("Have you tried the new chromuim based edge ?");
        }
        else
        {
            await this.next.Invoke(httpContext);
        }
    }
}
{% endhighlight %}

### Help this project ?

You can further help the project by visiting [http://bit.ly/detectbrowser](http://bit.ly/detectbrowser?techiesweb) in your browser and see the detection works. [File an issue](https://github.com/kshyju/BrowserDetector/issues/new) if you see wrong data.

Cheers


