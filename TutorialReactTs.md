# Create React Typescript application with .NET 6 API

## Create the React application

The simplest way to create a React Typesprict app is to use the "create-react-app" command

```
yarn create-react-app hello-app --template typescript
```

## Create the .NET 6 API web app

First create an empty API project

```
dotnet new web -n Api
```

Add the Spagme nuget package

```
dotnet add Api package Spagme
```

The .NET 6 default empty template should only contain a simple Program.cs class:

_Program.cs_

```c#
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```

### Define and create the API

We need to define the API interface and implementing class

_MyApi.cs_

```c#
namespace Api
{
    public interface IMyApi
    {
        Task<Message> Hello(string firstname, string lastname);
    }

    public class MyApi : IMyApi
    {
        public Task<Message> Hello(string firstname, string lastname)
        {
            return Task.FromResult(new Message()
            {
                Greeting = $"Hello, {firstname} {lastname}!"
            });
        }
    }

    public class Message
    {
        public string? Greeting { get; set; }
    }
}
```

### Add Spagme endpoint

_Program.cs_

```c#
using Spagme;
using Api;

//Create app builder
var builder = WebApplication.CreateBuilder(args);

//Dependency injection
builder.Services.AddTransient<IMyApi, MyApi>();

//Build
var app = builder.Build();

//Use routing
app.UseRouting();

//Add Spagme endpoint
app.UseEndpoints(endpoints =>
{
    //Spagme api endpoint
    endpoints.Map("myapi/{method}", async context =>
    {
        //Get which method is called
        var method = context.Request.RouteValues["method"]?.ToString();
        if (string.IsNullOrWhiteSpace(method))
        {
            context.Response.StatusCode = StatusCodes.Status400BadRequest;
            await context.Response.WriteAsync("method is null");
        }

        try
        {
            //Parse input parameters
            var input = context.Request.HasFormContentType
                ? context.Request.Form.ToDictionary(o => o.Key.ToLower(),
                 pair => pair.Value.ToString())
                : context.Request.Query.ToDictionary(o => o.Key.ToLower(),
                 pair => pair.Value.ToString());

            //Call the api
            var myapi = endpoints.ServiceProvider.GetService<IMyApi>();
            var resp = await SpagmeApi.Call(myapi, method, input);

            //Create response
            context.Response.StatusCode = StatusCodes.Status200OK;
            context.Response.ContentType = System.Net.Mime.MediaTypeNames.Application.Json;
            await context.Response.WriteAsync(resp);
            return;

        }
        catch (Exception exc)
        {
            //Internal server error
            var logger = endpoints.ServiceProvider.GetService<ILogger<MyApi>>();
            if(logger!=null) logger.LogError(exc, $"Error when calling method {method}");
            context.Response.StatusCode = StatusCodes.Status500InternalServerError;
            await context.Response.WriteAsync(exc.Message);
            return;
        }
    });
});

//Start application
app.Run();
```

## Create the API
