---
layout: post
title:  "What, when and how of AngularJS configuration blocks"
date:   2017-04-27 17:15:03+05:30
tags: angularjs
author: akshar
---
Two blocks of modules run during bootstrap process. By bootstrap process we mean that they are run before any directive code or before any controller code. They essentially run before any developer written code. The two blocks are:

- Configuration blocks
- Run blocks

Configuration blocks are added by using `.config()` on module. Example:

	angular
		.module("home")
		.config(function () {
		});

So the function which is passed to .config() is the configuration block.

Run blocks are added by using `.run()` on module. Example:

	angular
		.module("home")
		.run(function () {
		});

In this post, we will try answering the following:

- What is .config() and configuration block
- When can config() be avoided and same functionality be achieved with constant(), service() etc.
- When configuration block and config() is needed
- What is injectable in configuration blocks
- When does config() execute

### Configuration blocks

In configuration blocks, we can inject any provider. 

	angular
		.module("home")
		.config(["$provide", function ($provide) {
			// This will add a provider to providerCache which can be used to get a service.
			$provide.provider();
			// This will add a provider to providerCache which can be used to get a service.
			// Service code will be encapsulated in a provider
			$provide.service();
		}]);

Some other providers we can inject to .config() are `$locationProvider`, `$compileProvider`, `$filterProvider` etc. Essentially you can inject any provider.

#### Cannot inject services into configuration blocks

You cannot inject services into configuration blocks. Configuration blocks are meant for configuring your modules. And services are meant for writing business logic. There shouldn't be use of any business logic during configuration phase.

Try injecting a service into configuration block and it will fail.

	// This will fail because $http is a service
	angular
		.module("home")
		.config(["$http", function ($http) {
		}]);

	// This will fail because $compile is a service
	angular
		.module("home")
		.config(["$compile", function ($compile) {
		}]);

### When do we not need configuration blocks

In several applications you wrote, you might not have needed configuration blocks. Angular provides some helpers which avoids the need of explicit configuration blocks.

Providers, services and constants are registered during configuration phase, which is part of bootstrap phase. AngularJS provides following helper functions to register providers, services and constants:

* angular.module("somemodule").provider()
* angular.module("somemodule").service()
* angular.module("somemodule").constant()

Had these methods not been there on moduleInstance, then we would have needed configuration blocks.

Suppose we want to keep a constant in our application. This constant tells the project name which will be shown on various screens of the app. The usualy way to achieving it would be:

	angular.module("core", []);
	angular
		.module("core")
		.constant("APP_NAME", "Vizbi: The most powerful BI tool")

And then this constant can be used in any controller and can be made available on scope.

	angular.module("visualization", ["core"]);
	angular
		.module("visualization")
		.controller(["APP_NAME", function (APP_NAME) {
			console.log(APP_NAME);
		}]);

Let's try the same thing with configuration block instead of using `.constant()`.
Ensure that you delete the old code which was using `.constant()`.

	angular.module("core", []);
	angular
		.module("core")
		.config(["$provide", function ($provide) {
			$provide.constant("APP_NAME", "Vizbi: The most powerful BI tool")
		}]);

APP_NAME would still be injectable in controller.

Similarly we usually add a service using angular.module().service(). But the same thing can usually be achieved using $provide.service().

Under the hood angular.module().constant() uses $provide.constant(). Similarly angular.module().service() uses $provide.service().

So most of the times you aren't having to use configuration block because there are helper methods like `.constant()`, `.provider()`, `.service()`, `.directive()`, `.controller()` etc.

### When will you definitely need configuration block

Services are provided by providers. eg: $http is provided by $httpProvider. At low level serives are encapsulated into providers. It is possible that providers have some properties and the functionality of the service depends on a particular property. Usually all the properties of the provider has sane default configurations and services use these defaults. But if we want services to behave differently we need to overrirde these defaults. An example would make it clear.

A very commonly used service is $http. $http is provided by $httpProvider and you can see code for it <a href="https://github.com/angular/angular.js/blob/e23782b8c23fc766efb29a87a25bc054af3159fd/src/ng/http.js#L257" target="_blank">here</a>.

By default after every http response, the digest cycle runs which brings the view in sync with model. But digest cycle take a while to run. If you want performance improvements, you can tell $http to combine processing of multiple http responses and run a single digest cycle after several responses.

This can be achieved by setting a <a href="https://github.com/angular/angular.js/blob/e23782b8c23fc766efb29a87a25bc054af3159fd/src/ng/http.js#L358" target="_blank">configuration property</a> on $httpProvider.

So you would inject $httpProvider in your code and change this default.

	angular.module("core", []);
	angular
		.module("core")
		.config("$httpProvider", function ($httpProvider) {
			$httpProvider.useApplyAsync(true);
		});

So you would have to use .config() when you want to change configuration of a ngCore module which has already been written in angular.js or if you want to change configuration of a third party library which your app uses.

You would rarely have to write configuration block for modules or providers which you define.

### When is configuration block executed

Configuration block is executed during bootstrap process which is before any controller or directive code. Once bootstrap is done, configuration blocks can't execute. Once boostrap is done, providers aren't injectable any more. So property or method on providers can't be called after boostrap phase. This essentially means that property or methods of providers can't be called outside of configuration phase.



