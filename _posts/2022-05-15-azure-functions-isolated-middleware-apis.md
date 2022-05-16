---
layout: post
title: Useful APIs for writing azure function middleware
date: 2022-05-15 00:00:00.000000000 -05:00
tags: [Azure functions, Functions, Isolated azure functions, Middleware, Functions middleware, IFunctionsWorkerMiddleware, GetHttpRequestDataAsync, GetInvocationResult, GetHttpResponseData]
description: Useful APIs for writing azure function middleware.
---

One of the benefits of using isolated (out-of-process) model is the ability to plug in custom middlewares in the invocation pipeline. In this post, I will go through some of the handy APIs you can use to make your middleware authoring experience easier. All these APIs are extension methods on `FunctionContext` instance which is available in the `Invoke` method of the middleware. 



### HTTP Trigger specific APIs

Let us take a look at 2 APIs specific to Http triggers.

#### GetHttpRequestDataAsync

The `GetHttpRequestDataAsync` extension method can be used to obtain an instance of `HttpRequestData` when called during an http trigger invocation. In the isolated function model, the HttpRequestData instance represents the http trigger specific information, which is useful when you want to read/update http trigger specific data, such as request/response headers and cookies. The `GetHttpRequestDataAsync` returns a `ValueTask<HttpRequestData?>` instance, which you could await to get the `HttpRequestData` instance.

````
HttpRequestData? httpReqData = await functionContext.GetHttpRequestDataAsync();
````

This method will return `null` if called during a non http trigger invocation.

#### GetHttpResponseData
The `GetHttpResponseData` extension method can be used to get an instance of `HttpResponseData`  when called during an http trigger invocation. This method will return `null` if called during a non http trigger invocation or when the function return type is not `HttpResponseData`.

The following is an example of a middleware implementation which reads the HttpRequestData instance and updates the HttpResponseData instance during function execution. This middleware checks for the presence of a specific request header(`x-correlationId`), and when present uses the header value to stamp a response header. Otherwise, it generates a new GUID value and uses that for stamping the response header.

<script src="https://gist.github.com/kshyju/8b9c4a611f773369f14f40578dd74892.js?file=StampHttpHeaderMiddleware.cs"></script>

We can use the `UseWhen` extension method to register this middleware, where we can write a predicate which returns true, only when the invocation is for an http trigger. This way, our middleware will be participating in the invocation pipeline only for http trigger invocations. 

<script src="https://gist.github.com/kshyju/8b9c4a611f773369f14f40578dd74892.js?file=UseWhenExtensionUse.cs"></script>

### APIs applicable to all trigger types.

#### GetInvocationResult
The `GetInvocationResult` method gets an instance of `InvocationResult`, which represents the result of the current function execution. The `InvocationResult` type has a `Value` property to get or set the value as needed.

This method is useful when you want to alter the return value of the function inside a middleware. For example, you can write a global exception handler like below where you will update the return value when an exception happens during the invocation.

<script src="https://gist.github.com/kshyju/8b9c4a611f773369f14f40578dd74892.js?file=ExceptionHandlingMiddleware.cs"></script>

There is another overload of `GetInvocationResult` where you can specify the type you are expecting as the return type of the invocation.

````
 var invocationResult = context.GetInvocationResult<HttpResponseData>();
````

#### GetOutputBindings
The `GetOutputBindings` method gets the output binding entries collection for the current function execution. Each entry in the collection is of type `OutputBindingData<T>`. You can use the `Value` property to get or set the value as needed.

````
IEnumerable<OutputBindingData<object>>? allOutputBindings = context.GetOutputBindings<object>();
````

This method is useful if you want to update the invocation result of a function which has multiple output bindings.

For example, if you have a function like below where the return type of the function is a type which has properties decorated with output binding attributes.

<script src="https://gist.github.com/kshyju/8b9c4a611f773369f14f40578dd74892.js?file=HttpTriggerWithMultipleOutputBindings.cs"></script>


You can use the `GetOutputBindings` method to get the output binding entries and update them as needed. Below is an example where we are updating the value of the queue output binding entry.

<script src="https://gist.github.com/kshyju/8b9c4a611f773369f14f40578dd74892.js?file=GetOutputBindingsToUpdateValue.cs"></script>

#### BindInputAsync
The `BindInputAsync` method binds an input binding item for the requested BindingMetadata instance. For example, you can use this method when you have a function with a `BlobInput` input binding that needs to be accessed or updated by your middleware. Imagine you have a function like below,

<script src="https://gist.github.com/kshyju/8b9c4a611f773369f14f40578dd74892.js?file=HttpTriggerWithBlobInputFunction.cs"></script>


You can access all the input bindings for your function from the function context(via the `FunctionDefinition` property) and update the entry values. In the below snippet, I am updating only the blob input entry's value.


<script src="https://gist.github.com/kshyju/8b9c4a611f773369f14f40578dd74892.js?file=UpdateBlobInputBindingEntryValue.cs"></script>

Please give these APIs a try. These are available from version [1.8.0-preview1](https://www.nuget.org/packages/Microsoft.Azure.Functions.Worker/1.8.0-preview1) onwards. 


Cheers


