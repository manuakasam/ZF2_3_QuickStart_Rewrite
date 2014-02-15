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