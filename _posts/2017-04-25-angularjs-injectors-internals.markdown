---
layout: post
title:  "AngularJS injectors internals"
date:   2017-04-25 20:34:49+05:30
tags: angularjs
author: akshar
---
In this post we will try to answer the following questions.

* What is an injector?
* What functionality does an injector provide?
* Understanding different methods provided by injector.

#### What is an injector?

Injector is an object with some methods. It ensures that proper services are made available to the controllers by just using the name of the service.

Injector ensures that angular developers don't have to keep reference of services they create. Injector ensures that service references don't need to be passed around to every controller which needs services.

Assume you have a service in your app which is defined in home.service.js. It looks like:

	angular
		.module("home")
		.controller("HomeService", function () {
			var fetchProfileInfo = function () {
				return $http.get("someurl");
			}
			return {
				fetchProfileInfo: fetchProfileInfo
			};
		})

And we want to use this service in home.controller.js which looks like

	angular
		.module("home")
		.controller("HomeController", ["HomeService", function (HomeService) {
		}])

It's the job of the injector to ensure that HomeService is available inside HomeController so that HomeService.fetchProfileInfo() could be used from controller code.

Here reference of HomeService didn't have to be stored anywhere after creating the service. Also we didn't have to pass HomeService reference to controller. We were able to inject string "HomeService" in controller code and injector ensured that HomeService and it's methods are available inside controller.

There are many builtin services provided by Angularjs, like $compile, $http etc. It's the responsibility of the same injector to ensure these services are made available to controllers which need them.

#### What would happen if there was no injector

Had there been no injector, we would have had to keep reference to HomeService somehow. Probably something like the following.

	var HomeService = angular.module("somemodule").service();

Though this code snippet might not work because angular.module().service() doesn't return reference to the service. But had there been no injectors, angular.js code might have been written to return service reference when a service is created.

And then pass this reference to controller:

	angular.module("somemodule").controller("HomeController", function (HomeService) {
	});

This would have required both HomeService and HomeController to be defined in same file. Or else if we wanted to keep them in different files then we would have had to rely on function hoisting but this would have made the function avialable in global namespace, and hence would have polluted the global namespace.

#### When is the injector created.

Injector is created after all the scripts have loaded and when jqLite(document).ready() or $(document).ready() executes.

On document.ready, function <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/Angular.js#L1633" target="_blank">angularInit</a> is called. `angularInit` finds the html element which has `ng-app` defined on it. And then calls <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/Angular.js#L1663" target="_blank">function bootstrap</a> and passes this element having ng-app to bootstrap().

Injector is created inside this `bootstrap()`. boostrap() internally calls <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/Angular.js#L1757" target="_blank">createInjector()</a> to create the injector.

There are two injectors. Only 1 is exposed for angular developers to use. Other injector is internally used during bootstrap() and also internally used by the publically exposed injector.

That's why you would find many places which mention that there is only one injector. Because there is only one publically exposed injector.

#### Structure of injectors

createInjector() returns an injector. Internally createInjector() creates two injectors but returns only one injector. createInjector() ensures that only one injector is exposed.

Injectors are objects with following methods.

* <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/auto/injector.js#L862" target="_blank">invoke()</a>
* <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/auto/injector.js#L884" target="_blank">instantiate()</a>
* <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/auto/injector.js#L810" target="_blank">get()</a>
* <a href="" target="_blank">annotate()</a>
* <a href="" target="_blank">has()</a>

The two injectors created in createInjector() are:

	* providerInjector
	* instanceInjector

But only instanceInjector is exposed. providerInjector isn't exposed. providerInjector is used internally by angular during boostrap process. It's also used internally by instanceInjector during service discovery.

Both injectors have similar structure. An injector internally encapsulates a cache in which it stores the providers or services depending on which injector we are talking about.

providerInjector caches providers. instanceInjector caches services.

The injector cache ensure that only a singleton instance of service exists in the angular app. On first request for a particular service by any controller or any angular part, injector creates an instance of that particular service. After an instance of service has been created, it is cached in injector cache, and is only fetched from cache on every subsequent request.

#### providerInjector

Services under the hood are created by providers. Even when we call angular.module("somemodule").service(), under the hood a provider is created which encapsulates the service code. providerInjector caches the providers.

Every provider provides exactly one service and every service is provided by exactly one provider. It is like a one-to-one matching.

During injection process, instanceInjector is used. But this internally uses providerInjector to get the providers which in turn provides the related service.

providerInjector is created in createInjector() by calling <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/auto/injector.js#L664" target="_blank">createInternalInjector()</a>.

providerInjector's cache is named providerCache. It is a key value pair. Key is the name of provider and value is an object which has several methods providing provider functionality. providerCache caches following providers:
	* $provide
	* $injector (This is a reference to providerInjector, there is some magic involved. providerInjector is itself a provider too.)

These providers are created when providerInjector is created. These providers don't exist until providerInjector is created.

Both of these providers are added to providerCache in function `createInjector()`.

createInjector() then internally calls loadModules(). loadModules() iteratively runs `_invokeQueue` for all the registered modules, this happens in function <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/auto/injector.js#L762" target="_blank">runInvokeQueue()</a>. Some providers are added to the providerCache during `_invokeQueue` process. Some of them are:
	* Any provider added throught angular.module("somemodule").provider()
	* $compileProvider. This is added by using $provide.provider() from within config function of module ng.
	* $filterProvider
	* $anchorScrollProvider

And there are many more. You can see most of them <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/AngularPublic.js#L219" target="_blank">here, which happens during runInvokeQueue() of module `ng`.</a>

All these providers in turn provide some services.

#### instanceInjector

This is the injector which is <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/auto/injector.js#L685" target="_blank">returned</a> from `createInjector()`. This is the injector which is exposed and comes into play when controllers need access to a service. This is the **thing** which makes Dependency Injection possible.

This injector is stored as a data attribute on the element which has ng-app. You can see it <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/Angular.js#L1761" targt="_blank">here</a>.

So this injector remains reachable and usable after boostrap process too.

Controller functions are executed using <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/auto/injector.js#L862" target="_blank">`.invoke()`</a> of this injector. Injected service names are resolved as objects inside <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/auto/injector.js#L835" target="_blank">function injectionArgs()</a>. 

##### How the service names are resolved to proper objects

So whenever a particular service is needed by the controller function, injector looks for the service in it's cache, i.e in `injectorCache`. This happens in function <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/auto/injector.js#L810" target="_blank">getService()</a>. `getService()` is called from `injectionArgs()`.

getService() first checks the associated cache, which in this case is instanceCache. If service is already found in cache then it's just returned from cache. If service isn't found in cache, then the associated provider for this service is fetched from providerCache. And then service is retrieved from the provider and is cached on instanceCache. This ensures that next time serive is needed, there in no need to look for the associated provider in providerCache.


