---
layout: post
title: Azure functions V1 to V4 migration
date: 2022-10-16 00:00:00.000000000 -05:00
tags: [Azure functions, Functions, Isolated azure functions, Azure functions V1 to V4 migration, Azure functions migration]
description: Azure functions V1(in-proc) to V4(isolated) migration.
---

Azure functions dotnet isolated model recently started supporting  function app creation with .NET Framework as target framework. This is great news for devs maintaining V1 version of function apps because they can now migrate their V1 apps to the latest function runtime version(V4).

Let's take a look at the version history of function apps with the TFM & hosting models it supported.

| Function runtime 	| TFM support                                                                          	|
|------------------	|--------------------------------------------------------------------------------------	|
| V1               	| .NET framework, In-proc only                                                         	|
| V2               	| .Net core 2.X                                                                        	|
| V3               	| Out of process model is introduced. .NET 5<br>In-proc model with .Net core 3.X       	|
| V4               	| In-proc model with .Net 6<br>Out of process model is .Net6, .Net7 and .NET framework 	|


Until now, function apps created with V1 version could not migrate to the newer versions of functions runtime while maintaining the target framework, but it is possible now with the isolated model support of .NET framework TFM.

 > _The important thing to keep in mind is that, you will be migrating your in-proc app to isolated model._

### Migration fundamentals 

We will use a very simple V1 function app with Http trigger for the migration example.

### 1) Update the project file

Open your project file (.csproj) and make the below edits.

 - Update the `AzureFunctionsVersion` value from v1 to v4.
 - Add `OutputType` entry with value `Exe`.
 - Update PackageReferences.

    The in-proc model uses the `Microsoft.NET.Sdk.Functions` package. In the isolated model, we use a different set of packages.

    The below 2 packages are needed for all isolated function apps.
    - Microsoft.Azure.Functions.Worker
    - Microsoft.Azure.Functions.Worker.Sdk

    The below package is needed for Http functions. Since our example is migrating a function app with HttpTrigger, we need this package as well. In the in-proc model, the Microsoft.NET.Sdk.Functions implicitly brings the `Microsoft.Azure.WebJobs.Extensions.Http` package for http function support.

    - Microsoft.Azure.Functions.Worker.Extensions.Http

So after these changes, your .csproj file will look like this:

<script src="https://gist.github.com/kshyju/463f9adaa8fd4387b8e3ec1bd6d3b81a.js?file=v4-netfx-csproj-xml"></script>

<script src="https://gist.github.com/kshyju/8b9c4a611f773369f14f40578dd74892.js?file=UseWhenExtensionUse.cs"></script>


### 2) Updated configuration files

Open your `local.settings.json` file and add a new entry under `Values` with key `FUNCTIONS_WORKER_RUNTIME` and value `dotnet-isolated`

You can remove `AzureWebJobsDashboard` entry if present in your config. 

### 3) Add a Program.cs with startup code

Create a new class called `Program.cs` and paste the below code to it.

```
internal class Program
{
    static void Main(string[] args)
    {
        FunctionsDebugger.Enable();

        var host = new HostBuilder()
            .ConfigureFunctionsWorkerDefaults()
            .Build();

        host.Run();
    }
}
```

This is the bootstrapping code for the isolated application. It creates a generic host, configure the services needed for it to work as a function app, build the host and run it. 

### 4) Switch to the new types

The v1 in-proc version uses types from the `Microsoft.Azure.WebJobs` namespace. In the isolated model we do not use these types, instead a new set of types were introduced in the isolated model. So replace the using statements with `using Microsoft.Azure.Functions.Worker` from which we will use a few types.

 - In in-proc model, functions are decorated with the `FunctionName` attribute. In te isolated model, we will use the `Function` attribute.
 - In the Isolated model, the HttpRequestData type wraps information about the http request, instead of `HttpRequestMessage`
 - In the Isolated model, the `HttpResponseData` type should be used as the return type for Http functions.

 > _Ideally, An isolated function should not have any reference to the `Microsoft.Azure.WebJobs` package. Packages with `Microsoft.Azure.Functions.Worker` prefix is what you need in your isolated function app._

 https://www.nuget.org/packages?q=Microsoft.Azure.Functions.Worker


You should be good to run your app now. If you are using Visual studio, press F5 to start debugging.

Here are some useful links to help you with migration:

1. [Guide for running C# Azure Functions in an isolated process](https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide)
2. [Refer this sample solution](https://github.com/kshyju/NetFXMigrationSample) which has V1 in-proc project and V4 isolated version of the same.
2. [Isolated function app samples in the dotnet worker repo](https://github.com/Azure/azure-functions-dotnet-worker/tree/main/samples)
3. [Follow Azure functions on twitter](https://twitter.com/AzureFunctions)

If you are running into issues with the migration process, please open a new issue in the [dotnet worker repo](https://github.com/Azure/azure-functions-dotnet-worker/issues). 


Cheers


