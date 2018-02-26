---
title: .NET Core Configuration Loading in ASP.NET and Console Apps
subtitle: Configuration Consistency Across Apps
date: 2018-02-25 20:33:26
---

The packages [Microsoft.Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration/2.0.0) and [Microsoft.Extensions.Configuration.Json](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json/2.0.0) provide an easy way to load JSON configuration files into .NET Core applications. ASP.NET Core apps have these packages included by default and have some helpful paradigms that can be reused in Console apps.

I've created a couple example apps that showcase how to work with configuration loading in .NET Core. Check them out at my [BeerOnBeard GitHub repository exb-netcore-config](https://github.com/BeerOnBeard/exb-netcore-config).

ASP.NET Core apps use the environment variable `ASPNETCORE_ENVIRONMENT` to optionally load configuration files using the structure `appsettings.{ASPNETCORE_ENVIRONMENT}.json` as overrides to the values stored in `appsettings.json`. If the file exists, the system will load the file at startup.

Console apps in .NET Core do not reference the [Microsoft.Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration/2.0.0) and [Microsoft.Extensions.Configuration.Json](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json/2.0.0) packages by default. However, they can easily be added to projects to allow these console apps to use these helpful frameworks to load JSON configuration files and use the same optional overrides as ASP.NET Core apps using an environment variable with a little bit of extra code.

Settings files are not automatically shipped with Console apps when published, like they are in ASP.NET Core apps. In order to publish settings files to the output directory, add the following to the project's `.csproj` file. It will copy any file that starts with `appsettings`, such as `appsettings.json` and `appsettings.Development.json`, to the publish directory.

```xml
<Project Sdk="Microsoft.NET.Sdk">
...
  <ItemGroup>
    <Content Include="appsettings*" CopyToPublishDirectory="PreserveNewest" />
  </ItemGroup>
...
</Project>
```

The next code snippet will first set the base directory to the current working directory of the app where the settings files reside. Then, it will load in the `appsettings.json` file. Then, it will load in the settings file that is named using the environment variable, if it exists. The `optional` parameter lets the system know that the file might not exists and that it's not a big deal. `.Build()` is called to generate the final `IConfiguration` instance that can be used in the application.

```csharp
  var configuration = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json")
    .AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable("BEERONBEARD_ENVIRONMENT")}.json", optional: true)
    .Build();
```

In order to reduce potential collisions, a unique environment variable should be used to signify what configuration override to use. In this example, `BEERONBEARD_ENVIRONMENT` is used.

There are many ways to set environment variables. In my example apps, I show two ways to set the environment variables so they do not bleed out into the greater host OS. One is using VSCode build/debug using `launch.json`. The other uses Docker containers.

`launch.json` allows a user to specify environment variables to set when running an application locally. Check out the [OmniSharp documentation for more information](https://github.com/OmniSharp/omnisharp-vscode/blob/master/debugger-launchjson.md#environment-variables).

Dockerfiles also allow a user to specify environment variables that are available inside the running container. Check out the [Dockerfile env documentation for more information](https://docs.docker.com/engine/reference/builder/#env).

The new Microsoft rocks!
