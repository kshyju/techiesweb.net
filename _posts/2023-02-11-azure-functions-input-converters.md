---
layout: post
title: Extending input binding pipeline in dotnet isolated Azure functions 
date: 2023-02-11 00:00:00.000000000 -08:00
tags: [Azure functions, Functions, Isolated azure functions, Azure functions binding, Azure functions model binding]
description: Extending input binding pipeline in dotnet isolated Azure functions
---

The dotnet isolated model of azure functions has a binding pipeline which is responsible for populating your function inputs. While the built-in mechanism satisfies the majority of the binding use cases, there may be instances where you want to customize this pipeline. 

Let's get familiarized with a few terminologies before we move forward

### Input conversion

The process of using the raw input data to populate all the function inputs appropriately.

### Input converter

A small component responsible for converting the raw input to a specific type of function input type. The dotnet isolated package ships with a default set of converters and the Input conversion pipeline uses these converters to successfully bind all the function parameters.

## Extending the pipeline

If the built-in converters are not handling your use cases, you may write your own input converter and register that with your function app. Let's take a look at an example.

Imagine you have POCO like below to represent customer information.

```C#
public class Customer
{
    public string Name { get; set; }

    public string Gender { get; set; }

    public string Culture { get; set; }
}
```

and you want to use that type as a parameter of your function and you want to populate this from using some minimal information from your http request. For example, assume your http request includes the unique id of the customer in the route (ex: "`/api/customers/123`") and you want to use that id value to pull the customer details from another source (ex: a database /XML file/ a REST API etc..)

```C#
[Function("Update")]
public HttpResponseData Update(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get",Route ="customers/{id}")] HttpRequestData req,
    Customer customer)
{
    _logger.LogInformation($"Processing request for {customer.Id}");

    //do something with the customer instance.

    var response = req.CreateResponse(HttpStatusCode.OK);
    response.WriteString($"Processed {customer.Id} {customer.Name}");

    return response;
}

```

To make this happen, we will extend the input conversion pipeline with a custom converter.

### Writing a custom input converter

Create a class and have it implement the [IInputConverter](https://learn.microsoft.com/en-us/dotnet/api/microsoft.azure.functions.worker.converters.iinputconverter?view=azure-dotnet) interface. The `InputConverter` interface has a `ConvertAsync` method which we are going to implement with our custom logic.

```C#
public class CustomerConverter : IInputConverter
{
    public ValueTask<ConversionResult> ConvertAsync(ConverterContext context)
    {
        // we are going to replace this with actual implementation shortly.
        throw new NotImplementedException();
    }
}
```

The `ConvertAsync` method receives an instance of [ConverterContext](https://learn.microsoft.com/en-us/dotnet/api/microsoft.azure.functions.worker.converters.convertercontext?view=azure-dotnet), from which we can get information about the raw input data and the target type (the parameter type) we are going to bind to. In this specific example, we need the Id value from the request URL/route. We could access that from the BindingContext on the FunctionContext.


```C#
public async ValueTask<ConversionResult> ConvertAsync(ConverterContext context)
{
    if (context.FunctionContext.BindingContext.BindingData.TryGetValue("id", out var idObj))
    {
        var id = Convert.ToInt32(idObj);
        try
        {
            var customer = await GetCustomerFromRestAPIAsync(id);
            if (customer is not null)
            {
                return ConversionResult.Success(customer);
            }
        }
        catch(Exception ex)
        {
            return ConversionResult.Failed(ex);
        }
    }

    return ConversionResult.Unhandled();
}
```
Here we are reading the Id route param value and using that to populate a `Customer` instance. The `GetCustomerFromRESTAPIAsync` method is simply calling a REST API to get the customer details.

```c#
private async Task<Customer?> GetCustomerFromRestAPIAsync(int id)
{
    var url = $"https://anapioficeandfire.com/api/characters/{id}";

    return await httpClient.GetFromJsonAsync<Customer?>(url);
}
```

>  * _httpClient is a static instance of HttpClient._
>  * _The above method is kept as minimal and simple. Modify as needed to make it more robust as needed._


You can see that we used 3 helper methods in our `ConvertAsync` code. Let's take a look at what they do.

### ConversionResult.Success

 If the conversion is successful we return the result of `ConversionResult.Success` helper method, which returns an instance of `ConversionResult` with the `Status` property value set to `Succeeded` and `Value` property value set to the populated customer object. 

### ConversionResult.Failed

If there is was exception during the conversion, we use the  `ConversionResult.Failed` helper which returns a `ConversionResult` instance with the `Error` property populated. The `Status` property value will be set to `Failed` in this case.

### ConversionResult.Unhandled

 Finally the `ConversionResult.Unhandled` method can be used when you want to tell the calling pipeline that this converter is not able to convert this specific input. Remember this converter will be called while trying to populate every function input (_although there is a way to optimize this which you will see below_). For example, the date time converter returns Unhandled when the target type is not a DateTime type or the source is not string.

 https://github.com/Azure/azure-functions-dotnet-worker/blob/7d659ced053af41d58274ea4a36d2b5db2138750/src/DotNetWorker.Core/Converters/DateTimeConverter.cs#L16-L19



### Register the custom converter in the pipeline.

The custom converter can be hooked up to the function app in any of the following 3 approaches.

### 1) App bootstrapping time
Add the new converter to `WorkerOptions.InputConverters` while bootstrapping the function app. 

```c#
var host = new HostBuilder()
    .ConfigureFunctionsWorkerDefaults(workerOptions =>
    {
        workerOptions.InputConverters.Register<CustomerConverter>();
    })
    .Build();

await host.RunAsync();
```
The above approach will add the custom converter as the last entry in the `InputConverters` collection. The binding pipeline calls each converter sequentially in the order they are registered. So in this case the `CustomerConverter` will be called only after all the built-in converters are called and all of them returned an Unhandled result.

If you want the custom converter to be used earlier, you can use the `RegisterAt` method.

```C#
// Adds MyCustomConverter to the set of available (built-in) converters, but at index 3. 
workerOptions.InputConverters.RegisterAt<CustomerConverter>(3);
```

### 2) Type level
Decorate the type used as the parameter of the function with `InputConverter` attribute where you will define the custom converter type to be used.

```c#
[InputConverter(typeof(BookConverter))]
public class Book
{
    public string Name { set; get; }

    public string Isbn { set; get; }
}
```
and

```c#
[Function("GetBook")]
public HttpResponseData Update(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get",Route ="books/{id}")] HttpRequestData req,
    Book book)
{
    _logger.LogInformation($"Processing request for {book.Name}");

    //do something with the Book instance.

    var response = req.CreateResponse(HttpStatusCode.OK);
    response.WriteString($"Processed {book.Name}");

    return response;
}
```

_The above example assumes that you have written a `BookConverter` similar to the CustomerConverter above which populates a Book instance._

### 3) Function parameter level
Decorate the function parameter using the `InputConverter` attribute

```c#
[Function("GetBook")]
public HttpResponseData Update(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "books/{id}")] HttpRequestData req,
    [InputConverter(typeof(BookConverter))] Book book)
{
    _logger.LogInformation($"Processing request for {book.Name}");

    //do something with the Book instance.

    var response = req.CreateResponse(HttpStatusCode.OK);
    response.WriteString($"Processed {book.Name}");

    return response;
}
```
In this case, you do not need to decorate your book POCO with the `InputConverter` attribute.

```C#
public class Book
{
    public string Name { set; get; }

    public string Isbn { set; get; }
}
```

The last 2 approaches are more optimized than the first one since those will directly pick the explicitly specified converter for the parameter instead of going through all the registered converters.


### Dependency Injection

Yes, dependency injection work with your custom input converters. You can inject dependencies to your converter class constructor.

```c#
public class BookConverter : IInputConverter
{
    private readonly ILogger<BookConverter> _logger;

    public BookConverter(ILogger<BookConverter> logger)
    {
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
        
    public ValueTask<ConversionResult> ConvertAsync(ConverterContext context)
    {
        _logger.LogInformation($"BookConverter is the best!");
        throw new NotImplementedException();
    }
}
```



Give it a shot. If you are running into issues, please open a new issue in the [dotnet worker repo](https://github.com/Azure/azure-functions-dotnet-worker/issues). 


Cheers