---
layout: post
title: "From JSP to AngularJS"
description: "Moving from Server-Side templates to AngularJS"
category: "Software Development" 
tags: ["angularjs","spring mvc","jsp"]
---
{% include JB/setup %}

## Target Audience
This article is written for Spring developers who are familiar with JSP development and who would like to understand how to migrate to a frontend-based web application like AngularJS.

## Introduction on AngularJS

AngularJS is a Javascript framework created at Google that touts itself as a "Superheroic Web MVW Framework" (where the "W" in the "MVW" being a tongue-in-cheek reference to "Whatever" for all the various [MVx architecture](http://blogs.k10world.com/technology/difference-between-mvc-vs-mvp-vs-mvvm/)).As it is based on an MVx architecture, AngularJS provides a structure to Javascript development and thus gives Javascript an elevated status compared to traditional Spring + JSP application that probably uses Javascript to provide that bit of interactivity on the user interface. With AngularJS, your Javascript application will also inherit features like Dependency-Injection, HTML-vocabulary extension (via the use of custom directives), unit-testing and functional testing integration as well as DOM-selectors ala JQuery (using jqLite as it provides only a subset of JQuery but you could also easily use JQuery if you prefer). AngularJS also introduces scopes to your Javascript code so that variables declared in your code are bound only to the scope that is required.This prevents variables pollution that inadvertently arises when the size of your Javascript grows. 

Some of you who had used Javascript frameworks like Knockdown.js may recognise the value of automatic binding of a Javascript object/properties to a HTML view but AngularJS goes one step further and provide a mechanism for 2-way data-binding. That means that not only do you have the benefits of having your view updated with changes inside your Javascript model, any changes you make to your UI will also update the Javascript model (and consequently any other views that is bound to that model). It is almost magical to see all the views that are bound to the same JS model on the app update itself automatically. Moreover, since your model can be set to a particular scope, only views that belong to the same scope will be affected, allowing you to sandbox code that should be local only to a particular portion of your view. (This is done via an AngularJS attribute called ng-controller that is set in your HTML templates)

You can see an illustration of the difference here

![One-way Data Binding][one-way-binding]

![Two-way Data Binding][two-way-binding]

You can also see this in action with the following embedded plunker code

<iframe height="250px" width="100%" src="http://embed.plnkr.co/yAGCsQRH3M4LtysFy0fF/"></iframe>

Using AngularJS, it is possible to write relatively complex User Interface in an organised and elegant manner, always encapsulating the required logic within your components and never running the risk of errant global Javascript variables polluting your scope. As we can see later, it is also very testable, and there are built-in mechanisms to perform tests at the unit and functional level, ensuring that all your User Interface codebase goes through the same rigourous testing that your Java/Spring code undergoes, ensuring quality even at your user interfaces.  

## Differences in architecture between JSP and AngularJS

When one migrate from a server-side templating engine like JSP or [Thymeleaf](http://http://www.thymeleaf.org/ "thymeleaf.org") to a Javascript based templating engine,one has to make a paradigm shift towards a client-server architecture between the UI application that resides in the client (in this case, it could be a browser) and the server-side web services that abstracts the complexities of the server-side logic. 



## Considerations when moving from JSP to AngularJS
  - Application States
  - Authentication

## Practical Steps on migrating to AngularJS from JSP 


