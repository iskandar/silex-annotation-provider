silex-annotation-provider
=========================

A Silex ServiceProvider that defines annotations that can be used in a Silex controller.  Define your controllers in a class and use annotations to setup routes and define modfiers.


Installation
============

Install the silex-annotation-provider using composer. 

```
{
    "require": {
        "ddesrosiers/silex-annotation-provider": "dev-master"
    }
}
```

Parameters
==========
* **annot.srcDir**: The path to the silex-annotation-provider component.  This project uses Doctrine Annotations, provide this path to let silex-annotation-provider register the annotations for you.
* **annot.cache**: Speficy the type of Doctrine cache used to cache the annotations.  Instruct the reader to use the  {annot.cache}Cache class.  Make sure to include Doctrine Cache as it is not a required dependency of this project.
* **annot.controllers**: An array of fully qualified controller names.  If set, the provider will automatically register each controller as a ServiceController and set up routes and modifiers based on annotations found.

Services
========
* **annot**: The annot service includes a process() method that does the work of processing annotations and converting them to routes and modifiers.  Most won't ever need to use it directly.

Registering
===========
```php
$app->register(new DDesrosiers\SilexAnnotations\SilexAnnotationProvider(), array(
    "annot.srcDir" => __DIR__ . "/vendor/ddesrosiers/silex-annotation-provider/src",
    "annot.cache" => "Apc",
    "annot.controllers" => array("MyControllerNamespace\\MyController")
));
```

Usage
=====
The following is an example demonstrating the use of annotations to register an endpoint.
```
namespace DDesrosiers\Controller;

use DDesrosiers\SilexAnnotations\Annotations as SLX;
use Symfony\Component\HttpFoundation\Response;

class TestController 
{
	/**
	 * @SLX\Route(
	 *		@SLX\Request(method="GET", uri="test/{var}"),
	 *		@SLX\Assert(variable="var", regex="\d+"),
	 *		@SLX\RequireHttp,
	 *		@SLX\Convert(variable="var", callback="\DDesrosiers\Controller\TestController::converter")
	 * )
	 */
	public function testMethod($var)
	{
		return new Response("test Method: $var");
	}
	
	public static function converter($var)
	{
		return $var;
	}
}
```

The annotations above are equivalent to the following:
```
$app->get("test/{var}", "\\DDesrosiers\\Controller\\TestController:testMethod")
	->assert('var', '\d+')
	->requireHttp()
	->convert('var', "\\DDesrosiers\\Controller\\TestController::converter");
```
We assumed above that TestController was registered as a ServiceController.  If TestController is included in the annot.controllers array, it is automatically registered as a ServiceController and its annotations are automatically processed on boot.  

If we want to use a ControllerProvider, we can use the annotations service's process() method directly.

```
namespace DDesrosiers\Controller;

use DDesrosiers\SilexAnnotations\Annotations as Silex;
use Silex\Application;
use Silex\ControllerProviderInterface;
use Symfony\Component\HttpFoundation\Response;

class TestProviderController implements ControllerProviderInterface
{

	function connect(Application $app)
	{
		return $app['annot']->process(get_class($this), false, true);
	}

	/**
	 * @Silex\Route(
	 *		@Silex\Request(method="GET", uri="test/{var}"),
	 *		@Silex\Assert(variable="var", regex="\d+"),
	 * )
	 */
	public function testMethod()
	{
		return new Response("test Method");
	}
}
```

The ControllerProviderInterface's connect() requirement was satisfied by calling the annotation service's process() method.  process() takes 3 arguments:
* **controllerName**: The fully qualified class name of the controller to process.
* **isServiceController**: This matters because a Silex expect a different string representation of a controller method for ServiceControllers.  Default: false.
* **newCollection**: If true, all routes found will be put into a new controller collection and that collection will be returned.  Default: false.

Annotations
===========
**@Route**

The @Route annotation groups annotations into an isolated endpoint definition.  This is required if you have multiple aliases for your controller method with different modifiers.  All other annotations can be included as sub-annotations of @Route or stand on their own.

**@Request**

The @Request annotation associates a uri pattern to the controller method.  If multiple @Request annotations are given, all modifers will be applied to all @Requests unless they wrapped in a @Route annotation.
* method: A valid Silex method (get, post, put, delete, match)
* uri: The uri pattern.

**@Assert**

Silex\Route::assert()
* variable
* regex

**@Convert**

Silex\Route::convert()
* variable
* callback

**@Host**

Silex\Route::host()
* host

**@RequireHttp**

Silex\Route::requireHttp()

**@RequestHttps**

Silex\Route::requireHttps()

**@Value**

Silex\Route::value()
* variable
* default

**@Before**

Silex\Route::before()
* callback

**@After**

Silex\Route::after()
* callback
