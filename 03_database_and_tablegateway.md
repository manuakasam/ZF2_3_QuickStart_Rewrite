Working with the database backend
=================================

In the previous chapter we have created a `AlbumService` that returns some data from music-albums. While this served
an easy to understand learning purpose it is quite impractical for real world applications. No one would want to
modify the source files each time a new album is added. But luckily we all know about databases. All we need to learn
is how to interact with databases from our Zend Framework application. Let's start by creating a new Table inside the
database.

In this example we will focus on MySQL using PHP's PDO driver. Please create a database called `zf2tutorial` and run
the following SQL statements to create the album table including some default data.

```sql
CREATE TABLE album (
    id int(11) NOT NULL auto_increment,
    artist varchar(100) NOT NULL,
    title varchar(100) NOT NULL,
    PRIMARY KEY (id)
);
INSERT INTO album (artist, title)
    VALUES  ('The  Military  Wives',  'In  My  Dreams');
INSERT INTO album (artist, title)
    VALUES  ('Adele',  '21');
INSERT INTO album (artist, title)
    VALUES  ('Bruce  Springsteen',  'Wrecking Ball (Deluxe)');
INSERT INTO album (artist, title)
    VALUES  ('Lana  Del  Rey',  'Born  To  Die');
INSERT INTO album (artist, title)
    VALUES  ('Gotye',  'Making  Mirrors');
```

Feel free to modify the album and artist data to your liking!


Introducing the TableGateway
============================

Now that we have the database set up the big question is: "How to get to this data?". And there's many different ways
to approach this. We will be focusing on a pattern called "Table Data Gateway" and it's Zend Framework implementation
called [`Zend\Db\TableGateway\TableGateway`](https://github.com/zendframework/zf2/:current_branch/library/Zend/Db/TableGateway/TableGateway.php).

[Martin Fowler describes the `TableGateway`](http://martinfowler.com/eaaCatalog/tableDataGateway.html) as an object
that acts as a gateway to a database table. One instance handles all the rows in the table. Zend Framework has done a
good job in providing an implementation of this feature. We don't really have to write a lot of code to use it. All we
need to do is to use the existing classes and add some configuration to it. Let's create a Service called
`AlbumTableGateway`. Since we only need to use Zend Frameworks implementation and don't have to write a file at all, we
do this by adding and implementing the [`ServiceProviderInterface`](https://github.com/zendframework/zf2/:current_branch/library/Zend/ModuleManager/Feature/ServiceProviderInterface.php)
to our `Module` class. Just like with previous examples all you really need to do is to implement the `getServiceConfig()`
function as defined by the interface.

```php
<?php
// Filename: /module/Album/Module.php
namespace Album;

use Zend\Db\TableGateway\TableGateway;
use Zend\ModuleManager\Feature\AutoloaderProviderInterface;
use Zend\ModuleManager\Feature\ConfigProviderInterface;
use Zend\ModuleManager\Feature\ServiceProviderInterface;

class Module implements
    AutoloaderProviderInterface,
    ConfigProviderInterface,
    ServiceProviderInterface
{
    public function getAutoloaderConfig(){ /* ... */ }

    public function getConfig() { /* ... */ }

    /**
     * Expected to return \Zend\ServiceManager\Config object or array to
     * seed such an object.
     *
     * @return array|\Zend\ServiceManager\Config
     */
    public function getServiceConfig()
    {
        return array(
            'factories' => array(
                'AlbumTableGateway' => function($serviceLocator) {
                    $dbAdapter = $serviceLocator->get('Zend\Db\Adapter\Adapter');
                    return new TableGateway('album', $dbAdapter);
                }
            )
        );
    }
}
```

As you can see we have implemented the `getServiceConfig()` function and we have defined a Service factory called
`AlbumTableGateway`. The value of this service is an instance of the [`TableGateway`-Implementation]().
The `TableGateway` requires at least two parameters. The first parameter defines the name of the table inside our
database, which is `album`. The second parameter is the so-called `$dbAdapter` which basically is the connection to
the database itself. Zend Framework provides us with a DB-Adapter implementation, too, which listens to
`Zend\Db\Adapter\Adapter`.

Even if we're not done yet, let's see what happens if we try to use this TableGateway implementation at our
`AlbumService` right now.


AlbumTableGateway as Datasource
===============================

As mentioned within the previous chapter we want to use our `AlbumService` as single point of contact when dealing with
Album data. To make this happen we have to being the `AlbumTableGateway`-Service into the `AlbumService`. This is done
by using dependency injection the same way as we have done previously with our controller. Since now the `AlbumService`
will have a dependency we have to move it from the `invokables` into `factories` and create a factory class for this.
Start of by changing the configuration within `module.config.php`

```php
<?php
// FileName: /module/Album/config/module.config.php
return array(
    'service_manager' => array(
        'factories' => array(
            'Album\Service\AlbumService' => 'Album\Service\Factory\AlbumServiceFactory'
        )
    ),
    'view_manager' => array( /* ... */ ),
    'controllers' => array( /* ... */ ),
    'router' => array( /* ... */ )
);
```

Before you continue and create the `AlbumServiceFactory` please reload your application. When doing so you'll be
presented an exception that doesn't really seem to make sense.

```text
An error occurred
An error occurred during execution; please try again later.

Additional information:
Zend\ServiceManager\Exception\ServiceNotCreatedException

File:
{libraryPath}\Zend\ServiceManager\ServiceManager.php:{lineNumber}

Message:
An exception was raised while creating "Album\Controller\List"; no instance returned
```

Right now the `ServiceManager` lets you know that there was an exception while creating `Album\Controller\List`. All we
changed was the `AlbumService` though. This is due to the fact that `AlbumService` is a dependency of the
`Album\Controller\List` and due to the `AlbumServiceFactory` not existing this weird error happens. This becomes more
obvious when you notice that we actually have a previous exception displayed on the error page.

```text
Previous exceptions:
Zend\ServiceManager\Exception\ServiceNotCreatedException

File:
{libraryPath}\Zend\ServiceManager\ServiceManager.php:{lineNumber}

Message:
While attempting to create albumservicealbumservice(alias: Album\Service\AlbumService) an invalid factory was registered for this instance type.
```

The previous exception let's you know that the error is actually the alias `Album\Service\AlbumService` because we
have defined an invalid factory class for this alias. Let's fix this and create the factory class.

```php
<?php
// Filename: /module/Album/src/Album/Service/Factory/AlbumServiceFactory.php
namespace Album\Service\Factory;

use Album\Service\AlbumService;
use Zend\ServiceManager\FactoryInterface;
use Zend\ServiceManager\ServiceLocatorInterface;

class AlbumServiceFactory implements FactoryInterface
{
    public function createService(ServiceLocatorInterface $serviceLocator)
    {
        return new AlbumService();
    }
}
```

Reloading your application brings us back to our previous standing but that's actually not something we want. Now we
want to change the `AlbumService`-Class to use the `AlbumTableGateway` as a dependency. Go ahead and change the
`AlbumService` so that it requires an instance of a `TableGatewayInterface` within it's `__construct()`. Furthermore
you can delete the `$data`-property because we won't need it anymore.

```php
<?php
// Filename: /module/Album/src/Album/Service/AlbumService.php
namespace Album\Service;

use Album\Model\Album;
use Zend\Db\TableGateway\TableGatewayInterface;

class AlbumService implements AlbumServiceInterface
{
    protected $tableGateway;

    public function __construct(TableGatewayInterface $tableGateway)
    {
        $this->tableGateway = $tableGateway;
    }

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

Reloading the application would now reveal a fatal error telling us that we do have to give the `AlbumService` an
instance of the `TableGatewayInterface`.

```text
( ! ) Catchable fatal error: Argument 1 passed to Album\Service\AlbumService::__construct()
      must be an instance of Zend\Db\TableGateway\TableGatewayInterface, none given,
      called in /module/Album/src/Album/Service/Factory/AlbumServiceFactory.php on line 13
      and defined in /module/Album/src/Album/Service/AlbumService.php on line 12
```

Fix this by injecting the `AlbumTableGateway` into the `AlbumService` from within the factory `AlbumServiceFactory`

```php
<?php
// Filename: /module/Album/src/Album/Service/Factory/AlbumServiceFactory.php
namespace Album\Service\Factory;

use Album\Service\AlbumService;
use Zend\ServiceManager\FactoryInterface;
use Zend\ServiceManager\ServiceLocatorInterface;

class AlbumServiceFactory implements FactoryInterface
{
    public function createService(ServiceLocatorInterface $serviceLocator)
    {
        return new AlbumService(
            $serviceLocator->get('AlbumTableGateway')
        );
    }
}
```

Fixing one error should lead to another so reload the application and see what we get this time.

```text
An error occurred
An error occurred during execution; please try again later.

Additional information:
Zend\ServiceManager\Exception\ServiceNotFoundException

File:
{libraryPath}\library\Zend\ServiceManager\ServiceManager.php:{lineNumber}

Message:
Zend\ServiceManager\ServiceManager::get was unable to fetch or create an instance for Zend\Db\Adapter\Adapter
```

The `ServiceManager` doesn't know about the service `Zend\Db\Adapter\Adapter`. That's because we didn't define it yet.


Setting up the Database connection
==================================