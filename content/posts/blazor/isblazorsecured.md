---
title: "Is Blazor Web Assembly secured?"
date: 2020-06-03T16:25:23+10:00
draft: false
---
In this blog we will discuss about Blazor Web Assembly security at high level. Is Blazor Web Assembly secured?

Blazor allows Dlls to be downloaded to user browser. So user can get hold of the Dlls, decompile them and get source code from it. So at first go it might seem like a security issue, however it is not. We will discuss about it in detail, also few things or patterns that we need to be mindful when build application in Blazor.

## Reverse Engineer SPA from JavaScript

We have been using Angular, React and Vue JavaScript for long. These SPA frameworks and many other ones get compiled into JavaScript file and may be ugilified and minified. When user access this application JavaScript files are downloaded by user and they can get hold of the source code.

So it is possible to reverse engineer the application from JavaScript. Ofcourse it needs knowledge of JavaScript and the framework, but it is definitely possible.

## Reverse Engineer Blazor SPA from Dlls
