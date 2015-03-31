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

![One-way Data Binding](/assets/images/one-way-binding.png)

![Two-way Data Binding](/assets/images/two-way-binding.png)

You can also see this in action with the following embedded plunker code

<iframe height="250px" width="100%" src="http://embed.plnkr.co/yAGCsQRH3M4LtysFy0fF/"></iframe>

Using AngularJS, it is possible to write relatively complex User Interface in an organised and elegant manner, always encapsulating the required logic within your components and never running the risk of errant global Javascript variables polluting your scope. As we can see later, it is also very testable, and there are built-in mechanisms to perform tests at the unit and functional level, ensuring that all your User Interface codebase goes through the same rigourous testing that your Java/Spring code undergoes, ensuring quality even at your user interfaces.  

Another advantage of using AngularJS to write your html templates is that the templates are essentially similar to html even with the various frontend logic baked into your view. It is possible to incorporate AngularJS logic into your template and still pass a HTML validation. Try viewing a JSP file from a browser with all the template logic in place and most likely your browser will give up rendering the page. You can see how a typical AngularJS template looks like :

{% highlight html %}
<div class="row thumbnail-wrapper">
  <div data-ng-repeat="pet in currentOwner.pets" class="col-md-3">
    <div class="thumbnail">
      <img data-ng-src="images/pets/pet{{pet.id % 10 + 1}}.jpg" class="img-circle" alt="Generic placeholder image">
      <div class="caption">
        <h3 class="caption-heading" data-ng-bind="pet.name">Basil</h3>
        <p class="caption-meta" data-ng-bind="pet.birthdate">08 August 2012</p>
        <p class="caption-meta"><span class="caption-label" data-ng-bind="pet.type.name">Hamster</span></p>
      </div>
      <div class="action-bar">
        <a class="btn btn-default" data-toggle="modal" data-target="#petModal" data-ng-click="editPet(pet.id)">
          <span class="glyphicon glyphicon-edit" aria-hidden="true"></span> Edit Pet
        </a>
        <a class="btn btn-default" data-toggle="modal" data-target="#addPetVisitModal">
          <span class="glyphicon glyphicon-plus-sign" aria-hidden="true"></span> Add Visit
        </a>
      </div>
    </div>
  </div>
</div>
{% endhighlight %}

You can probably spot some non-HTML addition to the template includes attributes like "data-ng-click" which maps a click on a button to a method name call. There's also "data-ng-repeat" which will look through a JSON object and generate the necessary html to render the same view for each item in the list. Yet with all these logic in place, we are still able to validate and view the html template from the browser. AngularJS calls all the non-html tags and attributes "directives" and the purpose of these directives is to enhance the capabilities of HTML.

## Differences in architecture between JSP and AngularJS

When one migrates from a server-side templating engine like JSP or [Thymeleaf](http://http://www.thymeleaf.org/ "thymeleaf.org") to a Javascript-based templating engine, one has to experience a paradigm shift towards a client-server architecture. One has to cease thinking of the view as being a part of the web application and instead conceive the web application as 2 separate client-side and server-side applications. An illustration of this is in the next diagram which shows how a Spring application becomes a provider of RESTful Web Services, servicing various front end applications including an AngularJS browser-based application as well as a possibility to provide services for mobile clients like tablets or smartphones. These services could include OAuth, Authentication and other business logic services which should be obfuscated from public view. One should bear in mind that any data or business logic that is published in the form of JSON or javascript files are exposed for the client-side to see. Thus, if there's any business sensitive logic or workflow that should not be exposed, these logic should only be performed on the backend.

![AngularJS - Spring application](/assets/images/angularjs-spring.png)

## Considerations when moving from JSP to AngularJS

If you are considering migrating from JSP to AngularJS, there are a few factors to consider for a successful migration. 

### Converting your Spring controllers to RESTful services
You will need to transform your controllers so instead of forwarding the response to a tempting engine, you will provide services that will be serialised into JSON data instead. The following is an example of how a standard Spring MVC controller RequestMapping uses the ModelAndView object to render a view with the Owner as described in the url mapping.

{% highlight java %}
@RequestMapping("/owners/{ownerId}")
public ModelAndView showOwner(@PathVariable("ownerId") int ownerId) {
    ModelAndView mav = new ModelAndView("owners/ownerDetails");
    mav.addObject(this.clinicService.findOwnerById(ownerId));
    return mav;
}
{% endhighlight %}

A controller RequestMapping like that can be converted into an equivalent RESTful service that returns the owner based on the ownerId. Your template can then be moved into AngularJS which will then bind the owner object to the AngularJS template.

{% highlight java %}
@RequestMapping(value = "/owners/{id}", method = RequestMethod.GET)
public @ResponseBody Owner find(@PathVariable Integer id) {
    return this.clinicService.findOwnerById(id);
}
{% endhighlight %}

In order for Spring MVC to convert your return object (which needs to be Serializable) to a JSON object, you need to configure your Spring context file to include a conversion service that uses Jackson2 for JSON serialisation. An example of the snippet that performs this Spring context configuration is shown below.

{% highlight java %}
<bean id="objectMapper" class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean" p:indentOutput="true" p:simpleDateFormat="yyyy-MM-dd'T'HH:mm:ss.SSSZ"></bean>
<mvc:annotation-driven conversion-service="conversionService" >
	<mvc:message-converters>
		<bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter" >
			<property name="objectMapper" ref="objectMapper" />
		</bean>
	</mvc:message-converters>
</mvc:annotation-driven>
{% endhighlight %}

Once you have converted your controllers into RESTful services, you can then access these resources from your AngularJS application.

One nice way to access RESTful services in AngularJS is to use the built-in ngResource directive that allows you to access your RESTful services in an elegant and concise manner. An example of the Javascript code to access RESTful services using this directive can be illustrated by the following:

{% highlight javascript %}
var Owner = ['$resource','context', function($resource, context) {
	return $resource(context + '/api/owners/:id');
}];

app.factory('Owner', Owner);

var OwnerController = ['$scope','$state','Owner',function($scope,$state,Owner) {
	$scope.$on('$viewContentLoaded', function(event){
		$('html, body').animate({
		    scrollTop: $("#owners").offset().top
		}, 1000);
	});

	$scope.owners = Owner.query();
}];
{% endhighlight %}

The above snippet shows how a "resource" can be created by declaring an Owner resource and then initialising it as an Owner service. The controller can then use this service to query for Owners from the RESTful endpoint. In this way, you can easily create the resources that your application require and map it easily to your business domain model.

### Application States

An AngularJS application is essentially a Single Page Application from the perspective of the server side. It means that you may simply just serve a single JSP file as your main page and the AngularJS application will be responsible for displaying various view states thereafter. You will need to consider how you would like to detach your view logic from the JSP and transplant it onto AngularJS. One useful AngularJS plugin to consider for use is [UI-Router](https://github.com/angular-ui/ui-router/wiki) which provides a very good way to map view states to your location url. I prefer using this over the standard ngRoute module because UI-Router supports nested views (meaning states can be nested into more than just 1 level). An example on how you can configure the various location routes to map to the various states during the AngularJS application bootstrapping phase.

{% highlight javascript %}
app.config(['stateHelperProvider','$urlRouterProvider','$urlMatcherFactoryProvider',function(stateHelperProvider,$urlRouterProvider,$urlMatcherFactoryProvider) {

	$urlRouterProvider.otherwise("/");

	$urlMatcherFactoryProvider.strictMode(false)

	stateHelperProvider.state({
		name: "landing",
		url: "/",
		templateUrl: "components/landing/landing.html",
		controller: "MainController",
		data: { requireLogin : false }
	}).state({
		name: "dashboard",
		url: "/dashboard",
		templateUrl: "components/dashboard/dashboard.html",
		controller: "DashboardController",
		data: { requireLogin : true }
	}).state({
		name: "vets",
		url: "/vets",
		templateUrl: "components/veterinarians/veterinarians.html",
		controller: "VeterinarianController",
		data: { requireLogin : true }
	}).state({
		name: "pets",
		url: "/pets",
		templateUrl: "components/pets/pets.html",
		controller: "PetController",
		data: { requireLogin : true }
	}).state({
		name: "owners",
		url: "/owners",
		templateUrl: "components/owners/owners.html",
		controller: "OwnerController",
		data: { requireLogin : true }
	}).state({
		name: "ownerDetails",
		url: "/owners/:id",
		templateUrl: "components/owners/owner_details.html",
		controller: "OwnerDetailsController",
		data: {requireLogin : true}
	});

} ]);
{% endhighlight %}

You can read the UI-Router wiki to understand more about how this works.

### Authentication

If your web application requires authentication and authorization to resources or you implement some kind of user role management, you will need a way for your AngularJS application to authenticate users who login to your page. You might already be using Spring Security for your application and you can still reuse the same code with slight modifications on how you handle session state after authentication. This article does not go into details but you can refer to [this](https://spring.io/blog/2015/01/12/the-login-page-angular-js-and-spring-security-part-ii) for more information on how you can integrate Spring security and AngularJS to perform this function.

## Conclusion

Migrating to AngularJS from JSP may seem daunting but it can be very rewarding in the long run as it makes for a more maintainable and testable user interface. 

