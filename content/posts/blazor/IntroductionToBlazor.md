---
title: "Introuction to Blazor"
date: 2020-05-25T12:51:37+10:00
draft: false
---
Blazor Web Assembly is the new technology announced by Microsoft in Build conference 2020. This has been long time coming. 

## Why?

Dotnet has wide range of application frameworks available. It has Web Pages, ASP.NET MVC, Razor Pages as the main technologies. However other frameworks like Angular, React and Vue JS has become popular in last few years. These frameworks are built to be running on client browser as JavaScript. And they are really fast.

But Microsoft didn't had any framework that can run natively on client browser. Blazor Web assembly is the framework which will run on client browser.

React, Angular and Vue provide a component based programming model. It allows developer to create reusable components. There could be few stateful or stateless components. Dotnet ecosystem didn't had something like this. There is razor pages which can help in creating components but they compiled into html and served by the hosting server. Blazor is truely a component based programming model which doesn't depend on server to produce html. Rather html is produced on the browser with a combination of dotnet assemblies and wasm JavaScript.


## When?

First release was announced on May-2020 which is ready for Production  use.

Second release is going to be Nov-2020 with Dotnet version 5.

## What?

## The Aww Factor

There is something unique in this technology which is creating buzzz. It runs Dll on browser. Yes, when we run the application onto the browser all the Dlls are downloaded and these Dlls makes the application and are responsible for running the application in the browser. That is absolutely unique technology. Till date we had JavaScript running on the browser but now we can run assemblies compiled in various languages in the browser. This changes a lot of things. Few says its the end of JavaScript dominance in Single Page Application. But thats very hard to say and actual result will be when different companies will start adopting it.

This also mean that we can host this as a static website in say GitHub pages, Azure Static Website, Azure storage, CDN, S3 Bucket, etc.

How about first time load? Well the compiled version might be around 3-4 MB,which is not bad but still on the higher side. But good news is Microsoft team is working on it make the packaged version even smaller. May be better compression or something. 

## Conclusion

Blazor is definitely a promising technology and is going to make full stack development complete in Dotnet ecosystem. So we can build SPA, API, Jobs, Server side MVC, Windows, Progressive Web App, mamy more in single language and framework. Its truely one framework for all.
