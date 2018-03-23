---
layout: post
title:  "How script ordering works in an Angular app"
date:   2017-04-21 20:32:17+05:30
tags: angularjs
author: akshar
---
In this post, we will understand how ordering of scripts of an angular application make difference to the app.

In any angular application, there will be a lot of controllers, services, directives etc. Controllers would be dependent on services. Several times I found that in index.html, service.js is put after controller.js and I wondered how is controller able to get access to service if service is included after controller. I had several other questions like:

1. Do modules file need to be put before controllers?

2. Do services need to be put before controllers?

3. If services are put after controllers then why does the application behave properly. Why don't we get an error?

I will point to relevant angular.js code at various points. For this post, let's use angular 1.5.

### Javascript load and execution order

Javascript files are loaded and executed in the order they are encountered in html page. You can find more information at <a href="http://stackoverflow.com/questions/8996852/load-and-execute-order-of-scripts" target="_blank">Stack Overflow</a>.

So if there are two included scripts in index.html:

	<script src="first.js"></script>
	<script src="second.js"></script>

Then first.js would execute before second.js.

### angular.js must be included before any controller or service

In index.html, <script src="angular.js"></script> **must** appear before any angular specific thing. This needs to be done because any subsequent angular specific script would make use of object **angular**.

angular.js is the script which will ensure that any subsequent script has access to object `angular` or to method `angular.module`, or to method `angular.module("somemodule").controller()` etc.

As soon as angular.js executes, object `angular` is added to `window` and hence any subsequent script can make use of `angular`. angular.js also makes sure that a function called `module` is defined on object `angular`. And angular.js also makes sure that return value of `angular.module` has methods like `component`, `controller`, `service` etc. defined on them. All this happens inside function **setupModuleLoader**. You can see code of setupModuleLoader on <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/loader.js#L12" target="_blank">Github</a>

In any subseqent script, we are able define a module using `angular.module`. Also we are able to say `angular.module("somemodule").component()`. `angular.module("somemodule")` returns an object and this object has a method `component()` available on it. This object also has other methods like `service()`, `controller()`, `factory()`, `directive()` etc.

### Module should be defined before registering component, controllers etc.

Assuming you are going to create a controller called `HomeController` on module `home`. So script which defines home module must be included before script which defines the controller.

Assuming module is defined in home.module.js

	angular.module("home", [])

and controller is defined in home.controller.js

	angular
		.module("home")
		.controller("HomeController", function () {
		})

So script order must be:

	<script src="home.module.js"></script>
	<script src="home.controller.js"></script>

Similarly home.module.js must be included before home.service.js. But the load order of home.service.js or home.controller.js doesn't make any difference. Let's see the reason for this in next section.

### Needed services can be included after the controller which needs them

Assuming there is a service called HomeService defined in home.service.js and assuming HomeController needs the service.

home.service.js

	angular
		.module("home")
		.controller("HomeService", function () {
			return {
			};
		})

And assuming it is injected into HomeController, home.controller.js

	angular
		.module("home")
		.controller("HomeController", function (HomeService) {
		})

Doing angular.module("home").controller(...) does not run the function definition of controller. It only **registers** the controller. Registering a controller would become clearer as you read.

We are saying that load order of home.service.js and home.component.js doesn't make any difference.

It could be:

	<script src="home.module.js"></script>
	<script src="home.service.js"></script>
	<script src="home.controller.js"></script>

Or it could be:

	<script src="home.module.js"></script>
	<script src="home.controller.js"></script>
	<script src="home.service.js"></script>

#### How?

If you are an absolute beginner, you should skip this section.

Earlier we talked about <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/loader.js#L12" target="_blank">setupModuleLoader</a> which makes sure that object `angular` is available in subsequent scripts.

Inside setupModuleLoader, <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/loader.js#L26" target="_blank">this line</a> ensures that object angular has a function called `module` using which modules can be registered.

Internally a `module` is an object which has properties like `invokeQueue`, `configBlocks` etc. This object is created on <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/loader.js#L111" target="_blank">this line.</a> You will find that this object also has methods like `service`, `controller` etc.

When angular.module("mymodule").controller() executes, the function defining the controller gets stored in property invokeQueue. This happens in function <a href="https://github.com/angular/angular.js/blob/v1.5.x/src/loader.js#L352" target="_blank">invokeLaterAndSetModuleName</a>. This is what I meant when I said controller gets **registered**. The controller function doesn't execute immediately but is instead registered on invokeQueue. Since the controller function is only registered and isn't invoked, so there is no need for HomeService at this point.

Similarly when angular.module("mymodule").service() is executed, service is only registerd on invokeQueue . But service function isn't executed at this point.

As scripts are executed sequentially the services/controllers of those scripts are registered on invokeQueue. On document.ready, i.e after all scripts have loaded and executed, angular processes invokeQueue. During this the services become injectable. Angular ensures sure that controller function is only executed after processing invokeQueue of all modules. As the services are injectable at this point, if our code tries to inject a particular service into a controller, it is possible.

### Modules can be included in any order irrespective of which module needs which other module.

Assuming there is another module called "players" which has a controller which needs home services, i.e which is dependent on home.service.js.

Assuming players is defined in players.module.js

	angular.module("players", ["home"])

We specified "home" as required by "players" because players controller would be needing HomeService.

Assuming players controller is defined in players.controller.js

	angular
		.module("players")
		.controller("PlayerController", function (HomeService) {
		})

Even though players need home, it is not mandatory to include home before players. Script load order could be:

	<script src="players.module.js"></script>
	<script src="players.controller.js"></script>
	<script src="home.module.js"></script>
	<script src="home.controller.js"></script>
	<script src="home.service.js"></script>

And PlayerController would still be able to use HomeService defined in home.service.js.

This is possible because we specified `home` as **required** by `players`.

#### Internals

How is HomeService available in PlayerController even though it has been included after PlayerController script?

All scripts run sequentially. So module player was created and then PlayerController was registerd on it, in its invokeQueue. Function for PlayerController wasn't executed till that point. Then module home was created and then HomeService was registerd on it invokeQueue. Even function for HomeService wasn't executed till this point.

On document.ready, angular goes through all modules one by one and processes their invokeQueue. While processing the invokeQueue, angular makes the services injectable.

When angular tries to process module player, it finds that player is dependent on home. So it put player processing on hold and processes home. So invokeQueue of module `home` was processed and so HomeService became injectable. It is then available to be used by PlayerController.

### General rules

So the following general rules hold while including js scripts

* angular.js must be included before any module.js or component.js etc.
* module.js must be included before service.js, controller.js, component.js, directive.js etc.
* Different modules can be included in any order even if one module is dependent on another. So required module can be included after the requiring module. But you must ensure that `requires` argument is properly set.


