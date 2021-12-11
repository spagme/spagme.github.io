# Angular typescript app with .NET 6 minimal API

## Create the Angular application

The simplest way to create a Angular Typesprict app is to use the "ng new" command

```
ng new --style css --routing false hello-app
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
    SpagmeGen.Ts(typeof(IMyApi), "../hello-app/src/app/myapi.ts");

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

## Update Angular app

Create a new angular service for the api

Remember to update https://host with your host given to you by .NET

_src/app/myapi.service.ts_

```ts
import { Injectable } from "@angular/core";
import { Api, ApiBase } from "./myapi";

@Injectable({
  providedIn: "root",
})
export class ApiServiceImpl extends Api {
  constructor() {
    super("http://localhost:5000/myapi");
  }
}

@Injectable({
  providedIn: "root",
})
export abstract class ApiService extends ApiBase {}
```

Update module definition with a new provider for the api service

_src/app/app.module.ts_

```ts
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";
import { AppComponent } from "./app.component";
import { ApiService, ApiServiceImpl } from "./myapi.service";

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule],
  providers: [{ provide: ApiService, useClass: ApiServiceImpl }], //modify this line
  bootstrap: [AppComponent],
})
export class AppModule {}
```

Update the component class

_src/app/app.component.ts_

```ts
import { Component } from "@angular/core";
import { ApiService } from "./myapi.service";

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.css"],
})
export class AppComponent {
  greeting: string = "";

  constructor(private apiService: ApiService) {}

  ngOnInit() {
    this.apiService
      .hello("Mr", "President")
      .then((resp) => (this.greeting = resp?.greeting ?? ""));
  }
}
```

Update the component html file

_src/app/app.component.html_

```html
<h1>{{ greeting }}</h1>
```

## Run angular app

```
ng serve
```

Go to http://localhost:4200

![](TutorialAngularTs.png)
