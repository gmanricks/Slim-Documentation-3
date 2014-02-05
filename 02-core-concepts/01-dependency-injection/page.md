---
title: Slim Framework Documentation
---

Slim extends the [Pimple][1] DI container allowing you to abstract your classes dependencies using configurable factories & properties.

## Storing Properties

You can use Slim as a simple store, and attach different properties directly on the app object like so:

	<?php
	    $app = new \Slim\App();
	
	    $app['foo'] = 'bar';
	    echo $app['foo']; //=> bar

This provides a really easy way to attach static options or variables to the DI container. The downside with this approach is when you start using this to store instances of classes, everything gets instantiated immediately so if you don't end up using the variables then it wastes both processing time and memory to create all the settings.

For situations like these, you can assign a closure to create and return a new instance of the variable, that way it will only be stored in memory when you use it:

	<?php
	    $app = new \Slim\App();
	
	    class Bar {}; 
	
	    $app['foo'] = function () {
	        return new Bar(); 
	    };
	    
	    $bar = $app['foo'];

`$bar` will contain an instance of `Bar`, not the closure itself.

## Abstracting Dependencies

You can use these kinds of closure factories to abstract your dependencies in your classes, for example with the following class:

	<?php
	    class Bar {
	        public __construct(Logger $logger, $logLevel) {
	            $this->logger = $logger;
	            $this->logLevel = $logLevel;
	        }
	    }

With this code every time you want to instantiate a Bar object you will need to create a Logger and pass it in. Using Slim you can just create a factory and even use the DI container as a resource locator allowing the logger itself to be switched out:

	<?php
	   $app['level'] = 4;
	
	   $app['logger'] = function() {
	       return new Logger();
	   };
	
	   $app['bar'] = function($app) {
	       $logger = $app['logger'];
	       $level = $app['level'];
	       return new Bar($logger, $level);
	   };

The parameter passed into the the closure is the Slim app object, so you have access to all it's properties as-well as the other resources you attached to it.

## Singletons

If you simply assign a key in the container to a factory, then whenever you access that key a new instance will be created and returned. In situations where you want to have a singleton-like factory, you simply need to wrap the closure in a call to `$app->share`:

	<?php
	    $app = new \Slim\App();
	
	    class Bar {
	        public $val = 1;
	    }; 
	
	    $app['foo'] = $app->share(function () {
	        return new Bar(); 
	    });
	
	    $bar = $app['foo'];
	    $bar->val = 8;
	 
	    $qux = $app['foo'];
	    echo $qux->val; //=> 8

The first time the closure get's called it will run the provided closure, store the resulting instance and return it. On all subsequent calls to that key, it will simply return the already created instance, giving us a singleton-like effect.

## Storing Closures

In some situations you may just want to store a closure and have slim return the actual closure, not return the result of the closure, for example when storing a route handler. To do this you simply need to wrap the closure in a call to `$app->protect`:

	<?php
	    $app['world'] = $app->protect(function($name) {
	        echo "hello " . $name;
	    }); 
	 
	    $app->get('/hello/:world', $app['world']);

In the above code the actual closure will be returned much like a static property.

[1]:	http://pimple.sensiolabs.org/ "Pimple DI Container"
