Introducing Services and the ServiceManager
===========================================

In the previous chapter we've learned how to create a simple "Hello World" Application in Zend Framework 2. This is a
good start and easy to understand but the application itself doesn't really do anything. In this chapter we will
introduce you into the concept of Services and with this the introduction to [`Zend\ServiceManager\ServiceManager`](https://github.com/zendframework/zf2/blob/master/library/Zend/ServiceManager/ServiceManager.php).

What is a Service?
==================

//@todo someone who can define Service in a few sentences in easy to understand words

For what we're trying to accomplish with our `Album`-Module this means that we want to have a Service that will give
us the data that we want. The Service will get it's data from some source and when writing the Service we don't really
care about what the source actually is. The Service will be written against an `Interface` that we define and that
future Data-Providers have to implement.

Writing the AlbumService
========================

When writing a Service it is a common best-practice to define an `Interface` first. `Interfaces` are a good way to
ensure that other programmers can easily build extensions for our Services using their own implementations. In other
words, they can write Services that have the same function names but internally do completely different things but have
a similar result.

In our case we want to create an `AlbumService` so first we are going to define an `AlbumServiceInterface`. The task
of our Service is to provide us with data of our Albums. For now we are going to focus on the read-only side of things.
We will define a function that will give us all Albums and we will define a function that will give us a single Album.

Let's start by creating the Interface at `/module/Album/src/Album/Service/AlbumServiceInterface.php`

```php
<?php
// Filename: /module/Album/src/Album/Service/AlbumServiceInterface.php
namespace Album\Service;

interface AlbumServiceInterface
{
    /**
     * Should return a set of all albums that we can iterate over. Single entries of the array or \Traversable object
     * should be of type \Album\Model\Album
     *
     * @return array|\Traversable
     */
    public function findAllAlbums();

    /**
     * Should return a single album
     *
     * @param  int $id Identifier of the Album that should be returned
     * @return \Album\Model\Album
     */
    public function findAlbum($id);
}
```

As you can see we define two functions. The first being `findAllAlbums()` that is supposed to return all Albums and the
second one being `findAlbum($id)` that is supposed to return the Album matching the given identifier `$id`. What's new
in here is the fact that we actually define a return value that's not existing yet. We make the assumption that the
return value all in all are of type `Album\Model\Album`. We will define this class at a later point and for now we can
just create the `AlbumService` first.

Create the class `AlbumService` at `/module/Album/src/Album/Service/AlbumService.php`, be sure to implement the
`AlbumServiceInterface` and it's required functions. You then should have a class that looks like the following:

```php
<?php
// Filename: /module/Album/src/Album/Service/AlbumService.php
namespace Album\Service;

class AlbumService implements AlbumServiceInterface
{
    /**
     * @inheritDoc
     */
    public function findAllAlbums()
    {
        // TODO: Implement findAllAlbums() method.
    }

    /**
     * @inheritDoc
     */
    public function findAlbum($id)
    {
        // TODO: Implement findAlbum() method.
    }
}
```

Writing the required Model Files
================================

Since our `AlbumService` will return Models, we should create them, too. Be sure to write an `Interface` for the Model,
too! Let's create `/module/Album/src/Album/Model/AlbumInterface.php` and `/module/Album/src/Album/Model/Album.php`.
First we'll create the `Interface`:

```php
<?php
//filename: /module/Album/src/Album/Model/AlbumInterface.php
namespace Album\Model;

interface AlbumInterface
{
    /**
     * Will return the ID of the Album
     *
     * @return int
     */
    public function getId();

    /**
     * Will set the ID of the Album
     *
     * @param int $id
     */
    public function setId($id);

    /**
     * Will return the TITLE of the Album
     *
     * @return string
     */
    public function getTitle();

    /**
     * Will set the TITLE of the Album
     *
     * @param string $title
     */
    public function setTitle($title);

    /**
     * Will return the ARTIST of the Album
     *
     * @return string
     */
    public function getArtist();

    /**
     * Will set the ARTIST of the Album
     *
     * @param string $artist
     */
    public function setArtist($artist);
}
```

And now we'll create the appropriate Model file associated with the interface. Make sure to set the required class
properties and fill the getter and setter functions defined by our `AlbumInterface` with some useful content. You
then should have a class that looks like the following:

```php
<?php
// Filename: /module/Album/src/Album/Model/Album.php
namespace Album\Model;

class Album implements AlbumInterface
{
    /**
     * @var int
     */
    protected $id;

    /**
     * @var string
     */
    protected $title;

    /**
     * @var string
     */
    protected $artist;

    /**
     * @inheritDoc
     */
    public function getId()
    {
        return $this->id;
    }

    /**
     * @inheritDoc
     */
    public function setId($id)
    {
        $this->id = $id;
    }

    /**
     * @inheritDoc
     */
    public function getTitle()
    {
        return $this->title;
    }

    /**
     * @inheritDoc
     */
    public function setTitle($title)
    {
        $this->title = $title;
    }

    /**
     * @inheritDoc
     */
    public function getArtist()
    {
        return $this->artist;
    }

    /**
     * @inheritDoc
     */
    public function setArtist($artist)
    {
        $this->artist = $artist;
    }
}
```

Bringing Life into our AlbumService
===================================

Now that we have our Model files in place we can actually bring life into our `AlbumService` class. To keep the Service-
Layer easy to understand for now we will only return some static content from our `AlbumService` class directly. Create
a property inside the `AlbumService` called `$data` and make this an array of our Model type. Edit `AlbumService` like
this:

```php
<?php
// Filename: /module/Album/src/Album/Service/AlbumService.php
namespace Album\Service;

class AlbumService implements AlbumServiceInterface
{
    protected $data = array(
        array(
            'id'     => 1,
            'title'  => 'In  My  Dreams',
            'artist' => 'The  Military  Wives'
        ),
        array(
            'id'     => 2,
            'title'  => '21',
            'artist' => 'Adele'
        ),
        array(
            'id'     => 3,
            'title'  => 'Wrecking Ball (Deluxe)',
            'artist' => 'Bruce  Springsteen'
        ),
        array(
            'id'     => 4,
            'title'  => 'Born  To  Die',
            'artist' => 'Lana  Del  Rey'
        ),
        array(
            'id'     => 5,
            'title'  => 'Making  Mirrors',
            'artist' => 'Gotye'
        )
    );

    /**
     * @inheritDoc
     */
    public function findAllAlbums()
    {
        // TODO: Implement findAllAlbums() method.
    }

    /**
     * @inheritDoc
     */
    public function findAlbum($id)
    {
        // TODO: Implement findAlbum() method.
    }
}
```

After we now have some data, let's modify our `findXY()` functions to return the appropriate model files:

```php
<?php
// Filename: /module/Album/src/Album/Service/AlbumService.php
namespace Album\Service;

use Album\Model\Album;

class AlbumService implements AlbumServiceInterface
{
    protected $data = array(
        array(
            'id'     => 1,
            'title'  => 'In  My  Dreams',
            'artist' => 'The  Military  Wives'
        ),
        array(
            'id'     => 2,
            'title'  => '21',
            'artist' => 'Adele'
        ),
        array(
            'id'     => 3,
            'title'  => 'Wrecking Ball (Deluxe)',
            'artist' => 'Bruce  Springsteen'
        ),
        array(
            'id'     => 4,
            'title'  => 'Born  To  Die',
            'artist' => 'Lana  Del  Rey'
        ),
        array(
            'id'     => 6,
            'title'  => 'Making  Mirrors',
            'artist' => 'Gotye'
        )
    );

    /**
     * @inheritDoc
     */
    public function findAllAlbums()
    {
        $allAlbums = array();

        foreach ($this->data as $index => $album) {
            $allAlbums[] = $this->findAlbum($index);
        }

        return $allAlbums;
    }

    /**
     * @inheritDoc
     */
    public function findAlbum($id)
    {
        $albumData = $this->data[$id];

        $model = new Album();
        $model->setId($albumData['id']);
        $model->setTitle($albumData['title']);
        $model->setArtist($albumData['artist']);

        return $model;
    }
}
```

As you can see, both our functions now have appropriate return values. Please note that from a technical point of view
the current implementation is far from perfect. We will improve this Service a lot in the future but for now we have
a working Service that is able to give us some data in a way that we have defined by our `AlbumServiceInterface`.


Bringing the Service into the Controller
========================================

Now that we have our `AlbumService` written, we want to get access to this Service in our Controllers. For this task
we will step foot into a new topic called "Dependency Injection" short "DI".

// @todo Need someone to write a good 2-3 Sentences summary of what DI is in very easy to understand words

In our case we want to have our Album-Modules `ListController` somehow interact with our `AlbumService`. This means
that the class `AlbumService` is a dependency of the class `ListController`. Without the `AlbumService` our
`ListController` will not be able to function properly. To make sure that our `ListController` will always get the
appropriate dependency, we will first define the dependency inside the `ListControllers` constructor function
`__construct()`. Go on and modify the `ListController` like this:

```php
<?php
// FileName: /module/Album/src/Album/Controller/ListController.php
namespace Album\Controller;

use Album\Service\AlbumServiceInterface;
use Zend\Mvc\Controller\AbstractActionController;

class ListController extends AbstractActionController
{
    /**
     * @var \Album\Service\AlbumServiceInterface
     */
    protected $albumService;

    public function __construct(AlbumServiceInterface $albumService)
    {
        $this->albumService = $albumService;
    }
}
```

As you can see our `__construct()` function now has a required argument. We will not be able to call this class anymore
without passing it an instance of a class that matches our defined `AlbumServiceInterface`. If you were to go back to
your browser and reload your project with the url `domain.loc/album`, you'd see the following error message:

```text
( ! ) Catchable fatal error: Argument 1 passed to Album\Controller\ListController::__construct()
      must be an instance of Album\Service\AlbumServiceInterface, none given,
      called in {libraryPath}\Zend\ServiceManager\AbstractPluginManager.php on line {lineNumber}
      and defined in \module\Album\src\Album\Controller\ListController.php on line 15
```

And this error message is expected. It tells you exactly that our `ListController` expects to be passed an implementation
of the `AlbumServiceInterface`. So how do we make sure that our `ListController` will receive such an implementation?
The solution to this lies within our definition of our controller inside the `module.config.php`. Let's review:

```php
<?php
// FileName: /module/Album/config/module.config.php
return array(
    'view_manager' => array(/** ... */),
    'controllers' => array(
        'invokables' => array(
            'Album\Controller\List' => 'Album\Controller\ListController'
        )
    ),
    'router' => array(/** ... *)
);
```

As you can see our controller name `Album\Controller\List` is defined under the key `invokables`. Whenever you place
a class under the key `invokables` you let the [`Zend\Mvc\Controller\ControllerManager`](https://github.com/zendframework/zf2/blob/master/library/Zend/Mvc/Controller/ControllerManager.php)
know that it should call the class without wiring in any hard written dependencies. In other words: our `ListController`
can not be an `invokable` because it does have a hard written dependency called `AlbumServiceInterface`.

The [`Zend\Mvc\Controller\ControllerManager`](https://github.com/zendframework/zf2/blob/master/library/Zend/Mvc/Controller/ControllerManager.php)
does listen to another key, too, called `factories`. Factories are classes that are called from the Managers of the
Zend Framework Application. Every factory-class is supposed to have a return value. Usually this return value is a class.
Let's modify our configuration like this:

```php
<?php
// FileName: /module/Album/config/module.config.php
return array(
    'view_manager' => array(/** ... */),
    'controllers' => array(
        'factories' => array(
            'Album\Controller\List' => 'Album\Controller\Factory\ListControllerFactory'
        )
    ),
    'router' => array(/** ... *)
);
```

As you can see we no longer have the key `invokables`, instead we now have the key `factories`. Furthermore the value
of our controller name `Album\Controller\List` has been changed to not match the class `Album\Controller\ListController`
directly but to rather call a class called `Album\Controller\Factory\ListControllerFactory`. If you refresh your browser
you'll see a different error message:

```text
An error occurred
An error occurred during execution; please try again later.

Additional information:
Zend\ServiceManager\Exception\ServiceNotCreatedException

File:
{libraryPath}\Zend\ServiceManager\AbstractPluginManager.php:{lineNumber}

Message:
While attempting to create albumcontrollerlist(alias: Album\Controller\List) an invalid factory was registered for this instance type.
```

This message should be quite easy to understand. The [`Zend\Mvc\Controller\ControllerManager`](https://github.com/zendframework/zf2/blob/master/library/Zend/Mvc/Controller/ControllerManager.php)
is accessing `Album\Controller\List`, which internally is saved as `albumcontrollerlist`. While it does so it notices
that a factory class is supposed to be called for this controller name. However it doesn't find this factory class so
to the Manager it is an invalid factory. Using easy words: the Manager doesn't find the Factory class so that's probably
where our error lies. And of course, we have yet to write the factory, so let's go ahead and do this.


Writing a Factory Class
=======================

Factory classes within Zend Framework 2 always need to implement the [`Zend\ServiceManager\FactoryInterface`](https://github.com/zendframework/zf2/blob/master/library/Zend/ServiceManager/FactoryInterface.php).
Implementing this class let's the ServiceManager know that the function `createService()` is supposed to be called. And
`createService()` actually expects to be passed an instance of the `ServiceLocatorInterface` so the `ServiceManager` will
always inject this using Dependency Injection as we have learned above. Let's implement our factory class:

```php
<?php
// FileName: /module/Album/src/Album/Controller/Factory/ListControllerFactory.php
namespace Album\Controller\Factory;

use Album\Controller\ListController;
use Zend\ServiceManager\FactoryInterface;
use Zend\ServiceManager\ServiceLocatorInterface;

class ListControllerFactory implements FactoryInterface
{
    /**
     * Create service
     *
     * @param ServiceLocatorInterface $serviceLocator
     *
     * @return mixed
     */
    public function createService(ServiceLocatorInterface $serviceLocator)
    {
        $realServiceLocator = $serviceLocator->getServiceLocator();
        $albumService       = $realServiceLocator->get('Album\Service\AlbumService');

        return new ListController($albumService);
    }
}
```

Now this looks complicated! Let's start to look at the `$realServiceLocator`. When using a Factory-Class that will be
called from the ControllerManager it will actually inject itself as the `$serviceLocator`. However we need the real
ServiceManager to get to our ServiceClasses. This is why we call the function `getServiceLocator()` who will give us
the real ServiceManager.

//@todo Appendix link. read more information about different Manager classes and there factory implementations

After we have the `$realServiceLocator` set up we try to get a Service called `Album\Service\AlbumService`. This name
that we're accessing is supposed to return the matching Service that we need. This Service is then passed along to the
`ListController` which will directly be returned.

Note though, that we do have defined the class `Album\Service\AlbumService` but we did *not* register any Service
associated with this class yet. If you were to refresh your browser, the factory will be called but we will be greeted
with yet another new error message. Check it out!

```text
An error occurred
An error occurred during execution; please try again later.

Additional information:
Zend\ServiceManager\Exception\ServiceNotFoundException

File:
{libraryPath}\Zend\ServiceManager\ServiceManager.php:{lineNumber}

Message:
Zend\ServiceManager\ServiceManager::get was unable to fetch or create an instance for Album\Service\AlbumService
```

Exactly what we expected. Somewhere in our application - currently our factory class - a service called
`Album\Service\AlbumService` is requested but the ServiceManager doesn't know about this Service yet. Therefore it isn't
able to create an instance for the requested name.


Registering Services
====================

Registering a Service is as simple as registering a Controller. All we need to do is modify our `module.config.php` and
add a new key called `service_manager` that then has `invokables` and `factories`, too, the same way like we have it
inside our `controllers` array. Check out the new configuration file:

```php
<?php
// FileName: /module/Album/config/module.config.php
return array(
    'service_manager' => array(
        'invokables' => array(
            'Album\Service\AlbumService' => 'Album\Service\AlbumService'
        )
    ),
    'view_manager' => array(/** ... */),
    'controllers' => array(/** ... */),
    'router' => array(/** ... */)
);
```

As you can see we now have added a new Service. It may look weird since both strings are identical, but remember in what
way the configuration is actually set up. It's always a key=>value pair of `$serviceName => $serviceClass`. Since our
Service has no dependencies we are able to add this Service under the `invokables` array. Try refreshing your browser.
You should see no more error messages but rather exactly the page the we have created in the previous chapter of the
Tutorial.

Using the Service at our Controller
===================================

Let's now use the `AlbumService` within our `ListController`. For this we will need to overwrite the default
`indexAction()` and return the values of our `AlbumService` into the view. Modify the `ListController` like this:

```php
<?php
// FileName: /module/Album/src/Album/Controller/ListController.php
namespace Album\Controller;

use Album\Service\AlbumServiceInterface;
use Zend\Mvc\Controller\AbstractActionController;
use Zend\View\Model\ViewModel;

class ListController extends AbstractActionController
{
    /**
     * @var \Album\Service\AlbumServiceInterface
     */
    protected $albumService;

    public function __construct(AlbumServiceInterface $albumService)
    {
        $this->albumService = $albumService;
    }

    public function indexAction()
    {
        return new ViewModel(array(
            'albums' => $this->albumService->findAllAlbums()
        ));
    }
}
```

First please note the our controller imported another class. We need to import `Zend\View\Model\ViewModel`, which
usually is what your Controllers will return. When returning an instance of a `ViewModel` you're able to always
assign so called View-Variables. In this case we have assigned a variable called `$albums` with the value of whatever
the function `findAllAlbums()` of our `AlbumService` returns. In our case it is an array of `Album\Model\Album` classes.
Refreshing the browser won't change anything yet because we obviously need to modify our view-file to be able to display
the data we want to.


Accessing View Variables
========================

When pushing variables to the view they are accessible through two ways. Either directly like `$this->albums` or
implicitly like `$albums`. Both are the same however calling `$albums` implicitly will result in a little round-trip
through the `__call()` function which theoretically is a little slower. You won't ever notice this though.

Let's modify our view to display a table of all Albums we that our `AlbumService` returns.

```php
<h1>Albums</h1>

<table>
    <thead>
        <tr>
            <th>ID</th>
            <th>Artist</th>
            <th>Title</th>
        </tr>
    </thead>
    <tbody>
        <?php foreach($this->albums as $album) : ?>
        <tr>
            <td><?php echo $album->getId();?></td>
            <td><?php echo $album->getArtist();?></td>
            <td><?php echo $album->getTitle();?></td>
        </tr>
        <?php endforeach; ?>
    </tbody>
</table>
```

In here we simply define a little HTML-Table and then run a `foreach` over array `$this->albums`. Since every single
entry of our array is of type `Album\Model\Album` we can use the respective getter functions to receive the data we
want to get.

Summary
=======

And with this the current chapter is finished. We now have learned how to interact with the ServiceManager and we also
know what dependency injection is all about. We are now able to pass variables from our services into the view through
a controller and we know how to iterate over arrays inside a view-script.

If you have made it this far through the tutorial, the next chapter will be a breeze for you. We will simply implement
the functionality to display a single album instead of all albums. For you, this should be a piece of cake! Go ahead if
you're ready for it.