---
title: "Job scheduler with dotnet core"
date: 2020-05-10T15:54:26+10:00
draft: false
---


This is sample job scheduler with Topshelf, Autofac, Quartz, log4Net. Currently using DotNet Core 2.0 version

This repository implemetns a simple console application with following libraries

1. Topshelf : For creating service
2. Autofac : For dependency injection 
3. Quartz : For scheduling job, also uses xml based job configuration 
4. log4net : For logging 

## Installation 

Build the solution in release mode and copy to server directory. Open command prompt in the directory. 

Run below command : 

```SampleJobSchedulerWithDotNetCore.exe install -username “DOMAIN\Service Account” -password “Its A Secret” -servicename “SampleJobSchedulerWithDotNetCore” –autostart```


## Repo

You can find the repository here : 

https://github.com/svaus/SampleJobSchedulerWithDotNetCore

Create a dotnet core project. Add below packages by modifying csproj file or nuget package manager.

```
<ItemGroup>
    <PackageReference Include="Autofac" Version="4.9.1" />
    <PackageReference Include="Autofac.Extras.Quartz" Version="4.4.0" />
    <PackageReference Include="log4net" Version="2.0.8" />
    <PackageReference Include="Quartz" Version="3.0.7" />
    <PackageReference Include="Quartz.Jobs" Version="3.0.7" />
    <PackageReference Include="Quartz.Plugins" Version="3.0.7" />
    <PackageReference Include="Topshelf" Version="4.2.0" />
    <PackageReference Include="Topshelf.Autofac" Version="4.0.1" />
    <PackageReference Include="Topshelf.Log4Net" Version="4.2.0" />
    <PackageReference Include="Topshelf.Quartz" Version="0.4.0.1" />
  </ItemGroup>
```
Modify Program.cs file and add below code. This code is doing following 

1. Adding Log4Net for logging to file
2. Adding DI 
3. Initializing a HostFactory which sets windows service properties
4. Configure Log4Net with Topself

```
namespace SampleJobSchedulerWithDotNetCore
{
    class Program
    {
        static void Main(string[] args)
        {
            var logRepository = LogManager.GetRepository(Assembly.GetEntryAssembly());
            XmlConfigurator.Configure(logRepository, new FileInfo("log4net.config"));

            IContainer container = DependencyInjection.Build();


            HostFactory.Run(hostConfigurator =>
            {
                // Set windows service properties
                hostConfigurator.SetServiceName("SampleSchedulerService");
                hostConfigurator.SetDisplayName("Sample Service");
                hostConfigurator.SetDescription("Executes Job.");

                hostConfigurator.RunAsLocalSystem();
                // Configure Log4Net with Topself
                hostConfigurator.UseLog4Net();
                hostConfigurator.UseAutofacContainer(container);
                hostConfigurator.Service<SchedulerService>(serviceConfigurator =>
                {
                    serviceConfigurator.ConstructUsing(hostSettings => container.Resolve<SchedulerService>());
                    serviceConfigurator.WhenStarted(s => s.Start());
                    serviceConfigurator.WhenStopped(s => s.Shutdown());
                });
            });
        }
    }
}

```

It is very simple to create a job scheduler with dotnet core Hostfactory. On compilation this will generate an exe file which can just be added as windows service.
