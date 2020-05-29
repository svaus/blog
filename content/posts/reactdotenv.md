---
title: "Managing configuration in React App"
date: 2020-05-11T23:15:29+10:00
draft: false
---

Coming soon..

# Introduction 

React application these days is very easy to get started. However after using "getting started", few things get interesting. Found managing configuration in a React Application one of them. If you are trying with Create React App its different. But generally we dont use Create React App in actual real world application. May be some people do, but I won't. Why? Because I want little more control over dependencies, packages, extensibility, yada yada yada.

Managing configuration is one interesting scenario, I found in React App. It wasn't so out of the box thing for me. Managing configuration in an application that gets compiled and is available in clients browser, sounds wierd. Why do you even need configuration for such thing? Hang on!! Here comes DevOps. You should always build once and deploy as many times you like. It could be development, QA, Production environment. React App and SPA IN React is powerful, but ultimately is compiled(bundled) into one JavaScript file which is downloaded in the client browser.

