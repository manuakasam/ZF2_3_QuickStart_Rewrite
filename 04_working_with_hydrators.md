Working with Hydrators
======================

At the end of the last chapter we left with an error saying that the `ArrayObject` has no method `getId()`. And that's
more than logical because an `ArrayObject` doesn't really know what kind of potential properties it could have.
Accessing data from an `ArrayObject` is done using the `$object->$property` notation. In our example it would mean that
we don't call the `id` property via `$object->getId()` but rather through `$object->id`. This would totally work but it
isn't really something we **want**. Ultimately we want to work with our domain object `Album`. This means that we have
to figure out how to turn an `ArrayObject` into an `Album`-Object.

Luckily for us Zend Framework provides us with a functionality called `Zend\Stdlib\Hydrator` and is described as an act
of populating an object from a set of data. This is totally what we want! We do have a set of data, which is the
`ArrayObject`. And we want another Object, the `Album`-Object, filled with this data.

Zend Framework provides us with three different Hydrator-Implementations

- `Zend\Stdlib\Hydrator\ArraySerializable` //@todo link docs
- `Zend\Stdlib\Hydrator\ClassMethods`      //@todo link docs
- `Zend\Stdlib\Hydrator\ObjectProperty`    //@todo link docs

The hydrator of our choice is going to be the `ClassMethods`-Hydrator. This is due to the fact that our `Album`-Object
already has all its setters and getters defined and we don't really want to change anything within this class anymore.


Understanding the ClassMethods-Hydrator
=======================================

The way the `ClassMethods`-Hydrator works can be explained as follows:

1. The hydrator is instantiated
2. We give the hydrator some data in the form of an array
3. We tell the hydrator which object to populate
4. Form each given array entry it tries to call a setter-method from the object

To turn this written description into code take a look at the following example:

```php
$data = array(
    'id'     => 2,
    'artist' => 'Adele',
    'title'  => '21'
);

$hydrator = new \Zend\Stdlib\Hydrator\ClassMethods();

$album = $hydrator->hydrate($data, new Album());

\Zend\Debug\Debug::dump($album);

object(Album\Model\Album)#262 (3) {
  ["id":protected] => string(1) "1"
  ["title":protected] => string(14) "In  My  Dreams"
  ["artist":protected] => string(20) "The  Military  Wives"
}
```

As you can see we tell the `ClassMethods`-Hydrator to use it's `hydrate()` function. We tell it to use our array of
`$data` and populate this data into the object we're giving the hydrator `new Album()`. The result is a populated Object
of type `Album` with the data we want.


Using the Hydrator within the TableGateway
==========================================

Now that we know how the hydrator functions it's time to make use of it within our `AlbumTableGateway`. Zend Framework
once again provides us with all the tools we need and to turn our needs into productive code all we need to do is to
write 3 lines of code. Let's reflect on our current implementation of the `AlbumTableGateway`:

```php
<?php
// Filename: /module/Album/Module.php
namespace Album;

use Album\Model\Album;
use Zend\Db\TableGateway\TableGateway;
use Zend\ModuleManager\Feature\AutoloaderProviderInterface;
use Zend\ModuleManager\Feature\ConfigProviderInterface;
use Zend\ModuleManager\Feature\ServiceProviderInterface;

class Module implements
    AutoloaderProviderInterface,
    ConfigProviderInterface,
    ServiceProviderInterface
{
    public function getAutoloaderConfig() { /* ... */ }

    public function getConfig() { /* ... */ }

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

The way we're currently doing thins is to let the `TableGateway` do it's thing. All we tell the `TableGateway` is to
use the table called `album` and to use a `$dbAdapter` that we have defined. However the `TableGateway` has more options
that we can set inside it's `__construct()` function. The option of our interest is the fourth one called
`ResultSetInterface $resultSetPrototype`.

The `$resultSetPrototype`-parameter gives us the possibility to define what kind of `ResultSet` we're expecting the
`TableGateway` to return. Ultimately we want the `TableGateway` to give us `Album`-Objects. In the paragraphs above we
have learned that this is done through the method of hydration. The tool to bring these requirements together is called
`Zend\Db\ResultSet\HydratingResultSet` //@todo link it

The `HydratingResultSet` turns a `ResultSet` via a given hydrator into an object that we define. The `ResultSet` is
the default ReturnValue from the `TableGateway`. Let's now modify the `TableGateway` to use the `HydratingResultSet`
instead!

```php
<?php
// Filename: /module/Album/Module.php
namespace Album;

use Album\Model\Album;
use Zend\Db\ResultSet\HydratingResultSet;
use Zend\Db\TableGateway\TableGateway;
use Zend\ModuleManager\Feature\AutoloaderProviderInterface;
use Zend\ModuleManager\Feature\ConfigProviderInterface;
use Zend\ModuleManager\Feature\ServiceProviderInterface;
use Zend\Stdlib\Hydrator\ClassMethods;

class Module implements
    AutoloaderProviderInterface,
    ConfigProviderInterface,
    ServiceProviderInterface
{
    public function getAutoloaderConfig() { /* ... */ }

    public function getConfig() { /* ... */ }

    public function getServiceConfig()
    {
        return array(
            'factories' => array(
                'AlbumTableGateway' => function($serviceLocator) {
                    $dbAdapter = $serviceLocator->get('Zend\Db\Adapter\Adapter');

                    $resultSet = new HydratingResultSet();
                    $resultSet->setHydrator(new ClassMethods());
                    $resultSet->setObjectPrototype(new Album());

                    return new TableGateway('album', $dbAdapter, null, $resultSet);
                }
            )
        );
    }
}
```

So what did we do here? First we have created an instance of the `HydratingResultSet`. Then we have told it to make
use of the `ClassMethods`-Hydrator and populate the data into an `Album`-Object. Then we have configured the
`TableGateway` to use this `HydratingResultSet` and now we're at a stage where "it's done". Those three lines of code
were all we need to turn data from our database into an `Album`-Object.

Check your application and you'll see that the table of albums is rendered correctly again. No changes inside the view,
no changes inside our service, it's all done just by configuring the `TableGateway`.