---
layout: post
title: How to use Session variables in ASP.NET 5
date: 2015-12-07 00:11:29.000000000 -05:00
---
Using session in your ASP.NET 5 apps are a little tricky at first.
Since ASP.NET 5 has been redesigned to be so modular/configurable, you need to explicitly enable sessions in your app.



### Step 1
Project.json file is the central place where we define all the configuration options for the project. The dependecies section in this file
defines what other libraries/api's are being used in this project. You can think about each items in the dependecies section 
 as project references in projects from pre-ASP.NET 5 days.

To enable session, we need to include two items, `Microsoft.AspNet.Session` and `Microsoft.Extensions.Caching.Memory`. 
The great thing about editing project.js is the intellisense support from visual studio.
![project json changes](http://techiesweb.net/assets/project-json-with-dependencies-for-session.png)

Everytime you make a change to the file, Visual studio (dnu, behind the scene i guess) will restore the packages/dependecies definded in this file. 
(more like a nuget restore from packages.config)


### Step 2
Now we need to update the Startup.cs classe's `ConfigureServices` method where we will call `AddCaching` and `AddSession` methods.


    public void ConfigureServices(IServiceCollection services)
    {
        // Add framework services.

        // Added these 2 lines
        services.AddCaching();
        services.AddSession(s => s.IdleTimeout = TimeSpan.FromMinutes(30));

        services.AddMvc();
    }

	
You can see that i set the session timeout as 30 minutes.
	
Also we need to update the `Configure` method to call the `UseSession` method on the `IApplicationBuilder` instance

	public void Configure(IApplicationBuilder app, IHostingEnvironment env, 
	                                                  ILoggerFactory loggerFactory)
	{
	   // Your other existing code also goes here.
	   app.UseSession();
	   
	   app.UseMvc(routes =>
	   {
	 	routes.MapRoute(
				name: "default",
				template: "{controller=Home}/{action=Index}/{id?}");
	   });
	}

That is all. You should be good to use session variables in your ASP.NET 5 applications.


To set session,

     HttpContext.Session.SetInt32("SomeCounter",1)
      
To get session,

     var val = HttpContext.Session.GetInt32("SomeCounter");