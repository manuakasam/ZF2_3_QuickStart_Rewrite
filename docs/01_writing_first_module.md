Introducing our first "Album" Module
====================================

Now that we know about the basics of the Zend Framework 2 Skeleton Application, let's continue and create our very own
module. We will create a module named "Album". This module will display a list of database entries that represent a
single CD-Album. Each album will receive a couple of properties like `id`, `artist` and `title`. We will create forms
to enter new albums into our database and to edit existing albums. Furthermore we will do so by using best-practices
throughout the whole QuickStart..


Writing a new Module
====================

Let's start by creating a new folder under the `/module` directory called `Album`. Your directory structure should look
like this:

[[ DISPLAY IMAGE - DIRECTORY STRUCTURE /module/Album ]]

To be recognized as a module by ZF2s [ModuleManager](https://github.com/zendframework/zf2/blob/master/library/Zend/ModuleManager/ModuleManager.php)
all we need to do is create a PHP-Class named `Module` under our modules namespace, which is `Album`. Create the file
`/module/Album/Module.php` [[ LINK IMAGE - CURRENT DIRECTORY STRUCTURE /module/Album/Module.php ]]

```php
<?php
namespace Album;

class Module {}
```

We now have a module that can be detected by ZF2s [`ModuleManager`](https://github.com/zendframework/zf2/blob/master/library/Zend/ModuleManager/ModuleManager.php).
Let's try to add this module to our application. Even though it doesn't do anything yet, it should be able to be loaded
by the [`ModuleManager`](https://github.com/zendframework/zf2/blob/master/library/Zend/ModuleManager/ModuleManager.php).
To add this module to the application, add the entry `Album` to the file `/config/application.config.php` inside the
`modules` array.

```php
<?php
return array(
    // This should be an array of module namespaces used in the application.
    'modules' => array(
        'Application',
        'Album'         //@todo Highlight this line at .rts
    ),

    // These are various ......
    // .... all the other stuff remains the same
);
```

If you refresh your browser you should see no change at all, but no error message either. If that's the case then
continue to the next chapter. If you encounter any error, be sure to spell-check anything and follow the tutorial step
by step.


Configuring the Module
======================

Now that we have our module running it's time to start configuring it. To start of this process we will add a new
*route* to our application so that our module will be accessible through the url `domain.loc/album`. Achieving this
goal is done by letting the [`ModuleManager`](https://github.com/zendframework/zf2/blob/master/library/Zend/ModuleManager/ModuleManager.php)
know that our module actually does provide some configuration. This is done by adding and implementing the
[`ConfigProviderInterface`](https://github.com/zendframework/zf2/blob/master/library/Zend/ModuleManager/Feature/ConfigProviderInterface.php).
Continue by doing this to your `/module/Album/Module.php`

```php
<?php
namespace Album;

use Zend\ModuleManager\Feature\ConfigProviderInterface;

class Module implements ConfigProviderInterface
{
    public function getConfig()
    {
        return include __DIR__ . '/config/module.config.php';
    }
}
```

With this we let the [`ModuleManager`](https://github.com/zendframework/zf2/blob/master/library/Zend/ModuleManager/ModuleManager.php)
know that our module does provide some configuration for the application. However we are not done yet. The `getConfig()`
function expects to return an array or \\Traversable object. We will stick with the array version. And to further keep
our project organized in a well fashion we actually outsource this array into a separate file. Go ahead and create this
file under `/module/Album/config/module.config.php`:

```php
<?php
return array();
```

Try reloading your browser once and hopefully everything remains as it is and no error occurs. This is so because we
haven't actually added any configuration to our module yet. Let's finally get started and add the new route to our
module:

```php
<?php
return array(
    'router' => array(
        'routes' => array(
            'album-default' => array(
                'type' => 'Zend\Mvc\Router\Http\Literal',
                'options' => array(
                    'route'    => '/album',
                    'defaults' => array(
                        'controller' => 'Album\Controller\List',
                        'action'     => 'index',
                    ),
                ),
            )
        )
    )
);
```

With this we have created a new route called `album-default` that listens to the url `domain.loc/album`. Whenever
someone accesses this route a controller called `Album\Controller\List` will be accessed and more precisely its
`indexAction()`. The problem is that this controller does not yet exist and if you reload the page you will be greeted
with this lovely error message:

```text
A 404 error occurred
Page not found.
The requested controller could not be mapped to an existing controller class.

Controller:
Album\Controller\List(resolves to invalid controller class or alias: Album\Controller\List)
No Exception available
```

We now need to tell our module where to fine this controller named `Album\Controller\List`. To achieve this we have
to add this key to the `controllers` configuration key.

```php
<?php
return array(
    'controllers' => array(
        'invokables' => array(
            'Album\Controller\List' => 'Album\Controller\ListController'
        )
    ),
    'router' => array(
        // our previous route configuration
    )
);
```

The above configuration lets the application know that the controller key `Album\Controller\List` is meant as an alias
for the class `ListController` under the namespace `Album\Controller`. Sadly though this isn't enough yet. Reloading
the page results into yet another error:

```text
( ! ) Fatal error: Class 'Album\Controller\ListController' not found in {libPath}\Zend\ServiceManager\AbstractPluginManager.php on line {lineNumber}
```

This error tells us that the application know what class we want to access but sadly it simply cannot find the class.
That happens because our module never told the application where to find the classes that our module provides. To
have this information passed from our module to the application we need to add and implement the
[`AutoloaderProviderInterface`](https://github.com/zendframework/zf2/blob/master/library/Zend/ModuleManager/Feature/AutoloaderProviderInterface.php)
to our `Module` class.

```php
<?php
namespace Album;

use Zend\ModuleManager\Feature\AutoloaderProviderInterface;
use Zend\ModuleManager\Feature\ConfigProviderInterface;

class Module implements
    AutoloaderProviderInterface,
    ConfigProviderInterface
{
    public function getAutoloaderConfig()
    {
        return array(
            'Zend\Loader\StandardAutoloader' => array(
                'namespaces' => array(
                    __NAMESPACE__ => __DIR__ . '/src/' . __NAMESPACE__,
                )
            )
        );
    }

    public function getConfig()
    {
        return include __DIR__ . '/config/module.config.php';
    }
}
```

Now this looks like a lot of change but don't be afraid. The [`AutoloaderProviderInterface`](https://github.com/zendframework/zf2/blob/master/library/Zend/ModuleManager/Feature/AutoloaderProviderInterface.php)
defines that our module has to tell the application where to find its classes. For this we have a pretty easy
implementation of the `Zend\Loader\StandardAutoloader` that tells the application that all classes under the namespace
`__NAMESPACE__` (`Album`) are found inside the `/src` folder inside the current directory and then once again inside a
folder that's named like our namespace. So:

All classes that have the namespace `Album` are found inside `/module/Album/src/Album`. This holds true as long as we
hold on to [`PSR-0`](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md).

If you refresh the browser now you'll see that the error remains the same. And this is logical because remember what
we did by now. We do have told the application where to find the controller now, but we have yet to actually write the
controller itself. So let's continue with this. Let's create a controller class `Album\Controller\ListController`. To
reflect [`PSR-0`](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) we do separate namespaces into folders so you'll find this file under
`/module/Album/src/Album/Controller/ListController.php`:

```php
<?php
namespace Album\Controller;

class ListController {}
```

Reloading the page now will finally result into a new screen. The new error message looks like this:

```text
A 404 error occurred
Page not found.
The requested controller was not dispatchable.

Controller:
Album\Controller\List(resolves to invalid controller class or alias: Album\Controller\List)

Additional information:
Zend\Mvc\Exception\InvalidControllerException

File:
{libraryPath}\Zend\Mvc\Controller\ControllerManager.php:{lineNumber}
Message:
Controller of type Album\Controller\ListController is invalid; must implement Zend\Stdlib\DispatchableInterface
```

Explaining this goes beyond the scope of a QuickStart Tutorial but it is important that you see it. It basically tells
you that the application found something but has no idea how to deal with it. In MVC terms spoken: our application is
unable to dispatch the request through the controller we have given the application. We can solve this very easily by
having our controller extend the [`AbstractActionController`](https://github.com/zendframework/zf2/blob/master/library/Zend/Mvc/Controller/AbstractActionController.php).
Let's modify our controller now:

```php
<?php
namespace Album\Controller;

use Zend\Mvc\Controller\AbstractActionController;

class ListController extends AbstractActionController {}
```

It's now time for another refresh of the site. Big surprise, there's another error message for you.

```text
An error occurred
An error occurred during execution; please try again later.

Additional information:
Zend\View\Exception\RuntimeException

File:
{libraryPath}\library\Zend\View\Renderer\PhpRenderer.php:{lineNumber}
Message:
Zend\View\Renderer\PhpRenderer::render: Unable to render template "album/list/index"; resolver could not resolve to a file
```

Now the application tells you that a view template-file can not be rendered. Given our current progress this is more
than natural, because we have yet to actually write this view-file ourselves. The standard path would be
`/module/Album/view/album/list/index.phtml`. Create this file and add some dummy content to it:

```html
<h1>Album\ListController::indexAction()</h1>
```

Before we continue let us quickly take a look at where we placed this file. First off, view files are not to be found
under the `/src` directory because they are not source files. They are views so `/view` is much more logical. The
succeeding path however deserves some explanation but it's very simple. First we have the lowercased namespace. Following
by the lowercased controller name without the appendix 'controller' and lastly comes the name of the action that we are
accessing, again without the appendix 'action'. All in all it looks like this: `/view/{namespace}/{controller}/{action}.phtml`.
This has become a community standard but can potentionally be changed by you at any time.

However creating this file alone is not enough and this brings as to the final topic of this part of the QuickStart. We
need to let the application know where to look for view files. We do this within our modules configuration file `module.config.php`.

```php
<?php
return array(
    'view_manager' => array(
        'template_path_stack' => array(
            __DIR__ . '/../view',
        ),
    ),
    'controllers' => array(
        // our previous controllers configuration
    ),
    'router' => array(
        // our previous route configuration
    )
);
```

The above configuration tells the application that the folder `/module/Album/view` has view files in it that match the
above described default scheme. It is important to note that with this you can not only ship view files for your module
but you can also overwrite view files from other modules. You can find more information about this inside our Appendix
section of the QuickStart. //@todo Appendix link

//@todo performance hint template_map

Reload your site now. Finally we are at a point where we see something different than an error display. Congratulations,
not only have you created a simple "Hello World" kinda module, you also learned about many error messages and their
sources. If we didn't exhaust you too much, continue with our QuickStart and let's create a module that actually does
something.
