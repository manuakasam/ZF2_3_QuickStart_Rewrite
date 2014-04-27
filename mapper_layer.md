Preparing for different Database-Backends
=========================================

In the previous chapter we have created an `AlbumService` that returns some data from music-albums. While this served
an easy to understand learning purpose it is quite impractical for real world applications. No one would want to modify
the source files each time a new album is added. But luckily we all know about databases. All we need to learn is how
to interact with databases from our Zend Framework application.

But there is a catch. There are many database backend systems, namely SQL and NoSQL databases. While in a real-world
you would probably jump right to the solution that fits you the most currently, it is a better practice to create
another layer in front of the actual database access that abstracts the database interaction. We call this the
**Mapper-Layer**.


What is database abstraction?
=============================

The term "database abstraction" may sound quite confusing but this is actually a very simple thing. Consider a SQL and
a NoSQL database. Both have methods for CRUD operations. For example to query the database against a given row in
MySQL you'd do a `mysqli_query('SELECT foo FROM bar')`. But using an ORM for MongoDB for example you'd do something
like `$mongoODM->getRepository('bar')->find('foo')`. Both engines would give you the same result but the execution is
different.

So if we start using a SQL database and write those codes directly into our `AlbumService` and a year later we decide
to switch to a NoSQL database, we would literally have to delete all previously coded lines and write new ones. And
in a few years later a new thing pops up and we have to delete and re-write codes again. This isn't really the best
approach and that's precisely where database abstraction or the Mapper-Layer comes in handy.

Basically what we do is to create a new Interface. This interface then defines **how** our database interaction should
function but the actual implementation is left out. But let's stop the theory and go over to code this thing.


Creating the AlbumMapperInterface
=================================

Let's first think a bit about what possible database interactions we can think of. We need to be able to:

- find a single album
- find all albums
- insert new albums
- update existing albums
- delete existing albums

Those are the most important ones I'd guess for now. Considering `insert()` and `update()` both write into the database
it'd be nice to have just a single `save()`-function that calls the proper function internally.

Start by creating a new file inside a new namespace `Album\Mapper` called `AlbumMapperInterface.php` and add the
following content to it.

```php
<?php
// Filename: /module/Album/src/Album/Mapper/AlbumMapperInterface.php
namespace Album\Mapper;

use Album\Model\AlbumInterface;

interface AlbumMapperInterface
{
    /**
     * @param int|string $id
     * @return AlbumInterface
     * @throws \InvalidArgumentException
     */
    public function find($id);

    /**
     * @return array|AlbumInterface[]
     */
    public function findAll();

    /**
     * @param AlbumInterface $albumObject
     *
     * @param AlbumInterface $albumObject
     * @return AlbumInterface
     * @throws \Exception
     */
    public function save(AlbumInterface $albumObject);

    /**
     * @param AlbumInterface $albumObject
     *
     * @return bool
     * @throws \Exception
     */
    public function remove(AlbumInterface $albumObject);
}
```

As you can see we define four different functions. We say that a mapper-implementation is supposed to have one
`find()`-function that returns a single object implementing the `AlbumInterface`. Then we want to have one function
called `findAll()` that returns an array of objects implementing the `AlbumInterface`. Next we have the `save()`-
function to either **insert** or **update** the given album-object into the database and last but not least the
function `remove()` to delete the given album-object.


Refactoring the AlbumService
============================

Now that we have defined how our mapper should act we can make use of it inside our `AlbumService`. To start off the
refactoring process let's empty our class and delete all current content. Then implement the functions defined by the
`AlbumServiceInterface` and you should have an empty `AlbumService` that looks like this:

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
    }

    /**
     * @inheritDoc
     */
    public function findAlbum($id)
    {
    }
}
```

The first thing we need to keep in mind is that this interface isn't implemented in our `AlbumService` but is rather
used as a dependency. A required dependency, therefore we need to create a `__construct()` that takes any
implementation of this interface as a parameter. Also you should create a protected variable to store the parameter
into.

```php
<?php
// Filename: /module/Album/src/Album/Service/AlbumService.php
namespace Album\Service;

use Album\Mapper\AlbumMapperInterface;

class AlbumService implements AlbumServiceInterface
{
    /**
     * @var \Album\Mapper\AlbumMapperInterface
     */
    protected $albumMapper;

    /**
     * @param AlbumMapperInterface $albumMapper
     */
    public function __construct(AlbumMapperInterface $albumMapper)
    {
        $this->albumMapper = $albumMapper;
    }

    /**
     * @inheritDoc
     */
    public function findAllAlbums()
    {
    }

    /**
     * @inheritDoc
     */
    public function findAlbum($id)
    {
    }
}
```

With this we now require an implementation of the `AlbumMapperInterface` for our `AlbumService` to function. Since none
exists yet we can not get our application to work and we'll be greeted by the PHP error

```text
Catchable fatal error: Argument 1 passed to Album\Service\AlbumService::__construct()
must implement interface Album\Mapper\AlbumMapperInterface, none given,
called in {path}\module\Album\src\Album\Service\AlbumServiceFactory.php on line 19
and defined in {path}\module\Album\src\Album\Service\AlbumService.php on line 17
```

But the power of what we're doing lies within assumptions that we **can** make. This `AlbumService` will always have
a mapper passed as an argument. So in our `find*()`-functions we **can** assume that it is there. Recall that the
`AlbumMapperInterface` defines a `find($id)` and a `findAll()` function. Let's use those within our Service-functions:

```php
<?php
// Filename: /module/Album/src/Album/Service/AlbumService.php
namespace Album\Service;

use Album\Mapper\AlbumMapperInterface;

class AlbumService implements AlbumServiceInterface
{
    /**
     * @var \Album\Mapper\AlbumMapperInterface
     */
    protected $albumMapper;

    /**
     * @param AlbumMapperInterface $albumMapper
     */
    public function __construct(AlbumMapperInterface $albumMapper)
    {
        $this->albumMapper = $albumMapper;
    }

    /**
     * @inheritDoc
     */
    public function findAllAlbums()
    {
        return $this->albumMapper->findAll();
    }

    /**
     * @inheritDoc
     */
    public function findAlbum($id)
    {
        return $this->albumMapper->find($id);
    }
}
```

Looking at this code you'll see that we use the `albumMapper` to get access to the data we want. How this is happening
isn't the `AlbumService`s business anymore. But the `AlbumService` does know what data it will receive and that's the
only important thing.


The AlbumService has a dependency
=================================

Now that we have introduced the `AlbumMapperInterface` as a dependency for the `AlbumService` we are no longer able to
define this service as an `invokable` because it has a dependency. So we need to create a factory for the service. Do
this by creating a factory the same way we have done for the `ListController`. First change the configuration from an
`invokables`-entry to a `factories`-entry and assign the proper factory class:

```php
<?php
// Filename: /module/Album/config/module.config.php
return array(
    'service_manager' => array(
        'factories' => array(
            'Album\Service\AlbumServiceInterface' => 'Album\Factory\AlbumServiceFactory'
        )
    ),
    'view_manager' => array( /** ViewManager Config */ ),
    'controllers'  => array( /** ControllerManager Config */ ),
    'router'       => array( /** Router Config */ )
);
```

Going by the above configuration we now need to create the class `Album\Factory\AlbumServiceFactory` so let's go ahead
and create it:

```php
<?php
// Filename: /module/Album/src/Album/Factory/AlbumServiceFactory.php
namespace Album\Factory;

use Album\Service\AlbumService;
use Zend\ServiceManager\FactoryInterface;
use Zend\ServiceManager\ServiceLocatorInterface;

class AlbumServiceFactory implements FactoryInterface
{
    /**
     * Create service
     *
     * @param ServiceLocatorInterface $serviceLocator
     * @return mixed
     */
    public function createService(ServiceLocatorInterface $serviceLocator)
    {
        return new AlbumService(
            $serviceLocator->get('Album\Mapper\AlbumMapperInterface')
        );
    }
}
```

With this in place you should now be able to see the `ServiceManager` error message saying that the requested service
cannot be found.

```text
Additional information:
Zend\ServiceManager\Exception\ServiceNotFoundException
File:
{libraryPath}\Zend\ServiceManager\ServiceManager.php:529
Message:
Zend\ServiceManager\ServiceManager::get was unable to fetch or create an instance for Album\Mapper\AlbumMapperInterface
```

Conclusion
==========

We finalize this chapter with the fact that we successfully managed to keep the database-logic outside of our service.
Now we are able to implement different database solution depending on our need and change them easily when the time
requires it.

In the next chapter we will create the actual implementation of our `AlbumMapperInterface` using `Zend\Db\Sql`.