---
layout: post
title:  "Why AngularJS services aren't available in configuration blocks"
date:   2017-04-30 20:43:11+05:30
tags: angularjs
author: akshar
---
This post explains why services aren't available in configuration blocks.

AngularJS has a concept of $injector which has a function called invoke().

AngularJS has two injectors. They are:

- instanceInjector
- providerInjector

When we say $injector, most of the times we mean instanceInjector. You can read more about injectors in <a href="http://agiliq.com/blog/2017/04/angularjs-injectors-internals/" target="_blank">our last post</a>.

instanceInjector, or what we generally call $injector, can provide access to a service in either of the following ways:

- instantiate the service. Once instantiated instanceInjector caches the service
- get the service from cache if its alreay in cache

At low level, Controller or Service code is executed by $injector.invoke(). And this $injector knows a way to reach the services in one of the two ways mentioned above.

During configuration phase, i.e when configuration block is executed, $injector i.e instanceInjector isn't available. So instanceInjector cache isn't available too and so services aren't accessible in configuration blocks.

Try to access a service in configuration block and it will fail:

	angular.module("core", []);
	angular
		.module("core")
		.service("ProfileService", function () {
			var profile = {'name': 'hardcoded name'};
			return {
				'profile': profile;
			}
		});

Try to access this service in config block

	angular
		.module("core")
		.config(["ProfileService", function (ProfileService) {
		}]);

This failed because $injector isn't available when config block executed.

Also there shouldn't be a need to access services in configuration blocks. Configuration blocks are meant to configure things, eg: to configure a constant which will be used throughout the angular app. And services are meant to keep business logic. There shouldn't be need to use business logic during configuration phase.


