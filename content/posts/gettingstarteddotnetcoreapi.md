---
title: "Dotnet Core Api with Serilog, Autofac, Swagger"
date: 2020-05-10T13:06:28+10:00
draft: false
---

This blog post we will create a sample DotNet Core Api project with :

1. Serilog implementation with various sinks console, file, application insigths
2. Autofac
3. Swagger with ReDoc and Material Design

## Tools used

1. VS Code
2. Aspnet Core 2.2 SDK
3. Azure (For Application Insights only)


```cmd
dotnet new webapi
```

## Add Serilog Logging to project  

Add below packages to solution/project

```cmd
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Settings.Configuration
```

Modify the Program.cs file as below. Added method UseSerilog which is injecting log configuration from configuration file i.e. appsettings.json  

```C#

public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>()
                .UseSerilog((hostingContext, loggerConfiguration) => {
                    loggerConfiguration
                    .ReadFrom
                    .Configuration(hostingContext.Configuration)
                    .Enrich.FromLogContext()
                    .WriteTo.Console();
                });
```

Now add the below section in appsettings.json file at root level of appsetting json :

```C#
"Serilog": {
      "Using": [ "Serilog.Sinks.Console" ],
      "MinimumLevel": "Debug",
      "WriteTo": [
        { "Name": "Console" }        
      ],
      "Enrich": [ "FromLogContext", "WithMachineName", "WithThreadId" ],
      "Properties": {
      "Application": "SampleDotNetCoreApi"
      }
    }
```

In configure method of Startup.cs class add below code in the beginning and also inject ILoggerFactory. This ILoggerFactory is injected by AspNetCore dependency injection.

```C#
loggerFactory.AddSerilog();
```

### Add logging to Rolling File using Serilog

If you want to do logging to file, Serilog provides sink for RollingFile.

```cmd
dotnet add package Serilog.Sinks.RollingFile
```

And to appsettings.json file add the section for RollingFile. There is no code required for this. Serilog will read the configuration and apply the settings. The only thing which is required is adding the package.

```C#
"WriteTo": [
        { "Name": "Console" },
        {
        "Name": "RollingFile",
          "Args": {
          "pathFormat": "logs\\log-{Date}.txt",
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level}] {Message}{NewLine}{Exception}",
          }
        }
      ],
```

### Adding Application insight to Serilog

Adding application insights with Serilog is just two steps as below : 

```cmd
dotnet add package Serilog.Sinks.ApplicationInsights
```

Also add the below configuration in appsettings.json. Change the instrumentation key.

```C#
"WriteTo": [
        { "Name": "Console" },
        {
        "Name": "RollingFile",
          "Args": {
          "pathFormat": "logs\\log-{Date}.txt",
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level}] {Message}{NewLine}{Exception}",
          }
        },
        {
          "Name": "ApplicationInsights",
          "Args": {
            "instrumentationKey": "KEY",
            "telemetryConverter": "Serilog.Sinks.ApplicationInsights.Sinks.ApplicationInsights.TelemetryConverters.TraceTelemetryConverter, Serilog.Sinks.ApplicationInsights"
          }
        }
      ],

```

### Prefix for Development APM

Prefix is a free tool provided for development environment monitoring. Download the tool from below location 

[Prefix Download](https://stackify.com/prefix-download/)

Once downloaded install the tool in the development machine.

### Why add Autofac

Aspnet Core comes up with default dependency injection hence third party dependency injection is not required. However, default dependency injection doesn't have full functionality as offered by other DI libraries. Autofac is another DI library with lot of features built into it.

### Add Autofac

Insatall the package using below command. I am going to use without configure container method as usually in complex project we would need a lot of customisation and without configure container works in that scenario.

```cmd

dotnet add package Autofac.Extensions.DependencyInjection
```

For more details about Autofac check the documentation here -> [Autofac](https://autofaccn.readthedocs.io/en/latest/integration/aspnetcore.html)

Add a public property in the Startup.cs

```C#
public IContainer ApplicationContainer { get; private set; }
```

and add the below lines of code in ConfigureServices method and change the return type from void to IServiceProvider which will be replaced by new AutofacServiceProvider. This registers all the dependencies in that service to Autofac : 

```C#
            var builder = new ContainerBuilder();

            builder.Populate(services);

            this.ApplicationContainer = builder.Build();

            return new AutofacServiceProvider(ApplicationContainer);

```

### Add Swagger/Swashbuckle to API for documentation 


```cmd
dotnet add package Swashbuckle.AspNetCore
```

Add the below lines of code in ```ConfigureService``` method

```C#
            services.AddSwaggerGen(c =>
            {
                c.SwaggerDoc("v1", new Info{
                   Title = "Sample DotNet Core Api",
                   Version = "v1"
                });
            });
```

Add below lines in Configure method :

```C#
            app.UseSwagger();

            app.UseSwaggerUI(c=> {
                c.SwaggerEndpoint("/swagger/v1/swagger.json", "Sample DotNet Api V1");
                c.RoutePrefix = string.Empty;
            });
```

### Adding ReDoc for swagger for different UI theme

```cmd
dotnet add package Swashbuckle.AspNetCore.ReDoc
```

Add below code and comment out UseSwaggerUI piece of code

```C#
            // app.UseSwaggerUI(c=> {                
            //     c.SwaggerEndpoint("/swagger/v1/swagger.json", "Sample DotNet Api V1");
            //     c.RoutePrefix = string.Empty;
            //     c.InjectStylesheet("/swagger-ui/custom.css");
            // });

            app.UseReDoc(c => {
                c.SpecUrl = "/swagger/v1/swagger.json";
                c.DocumentTitle = "Sample DotNet Core Api V1";
            });

```

![ReDocUI]("https://github.com/svaus/SampleDotNetCoreApi/blob/master/ReDocUI.PNG")

### Add another custom style

Updated the custom.css for material design UI

![SwaggerMaterialUI]("https://github.com/svaus/SampleDotNetCoreApi/blob/master/SwaggerWithStyleChanges.PNG")
