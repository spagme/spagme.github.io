# React javascript app with .NET 6 minimal API

## Create the React application

The simplest way to create a React Typesprict app is to use the "create-react-app" command

```
npx create-react-app hello-app
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

//Create app
var builder = WebApplication.CreateBuilder(args);

//Set up services
builder.Services.AddTransient<IMyApi, MyApi>();
builder.Services.AddCors();

//Build app
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    //Generate client file
    SpagmeGen.Js(typeof(IMyApi), "../hello-app/src/myapi.js");

    //Allow CORS on localhost
    app.UseCors(x => x.SetIsOriginAllowed(origin => origin.ToLower().Contains("localhost")));
}

//Use routing
app.UseRouting();

//Add Spagme endpoint
//The endpoint will be: http(s)://<host>/myapi
app.UseSpagmeEndpoint(typeof(IMyApi), "myapi");

//Start application
app.Run();
```

## Update React app

Remember to update https://host with your host given to you by .NET

_App.js_

```jsx
import { useEffect, useState } from "react";
import "./App.css";
import { Api } from "./myapi";

function App() {
  const [greeting, setGreeting] = useState("");

  useEffect(() => {
    var api = new Api();
    api.url = "https://host/myapi";
    api.hello("Mr", "President").then((o) => {
      setGreeting(o?.greeting ?? "");
    });
  }, []);

  return (
    <div className="App">
      <header className="App-header">
        <p>{greeting}</p>
      </header>
    </div>
  );
}

export default App;
```

## Run react App

```
yarn start
```

Go to http://localhost:3000

![](TutorialReactTs1.png)
