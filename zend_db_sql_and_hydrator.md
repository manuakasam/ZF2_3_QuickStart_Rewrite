Introducing Zend\Db\Sql
=======================

In the last chapter we have introduced the mapping layer and created the `AlbumMapperInterface`. Now it is time to
create an implementation of this interface so that we can make use of our `AlbumService` again. As an introductionary
example we will be using the `Zend\Db\Sql` classes. So let's jump right into it.


Quick Facts Zend\Db\Sql
=======================

To create queries against a database using `Zend\Db\Sql` you need to have a database connection available. This
connection is served through any class implementing the `Zend\Db\Adapter\AdapterInterface`. The most handy way to
create such a class is through the use of the `Zend\Db\Adapter\AdapterServiceFactory` which listens to the config-key
`db`. Let's start by creating the required configuration entries and modify your `module.config.php` adding a new top-
level key called `db`:

```php
<?php
// Filename: /module/Album/config/module.config.php
return array(
    'db' => array(
        'driver'         => 'Pdo',
        'username'       => 'root',
        'password'       => 'admin',
        'dsn'            => 'mysql:dbname=album;host=localhost',
        'driver_options' => array(
            \PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES \'UTF8\''
        )
    ),
    'service_manager' => array(/** ... */),
    'view_manager'    => array(/** ... */),
    'controllers'     => array(/** ... */),
    'router'          => array(/** ... */)
);
```

As you can see we've added the `db`-key and inside we create the parameters required to create a driver instance. One
important thing to note is that in general you **do not** want to have your credentials inside the normal configuration
file but rather in a `/config/autoload/db.local.php` which will **not** be pushed to servers using the zend-skeletons
`.gitignore` file. Keep this in mind when you share your codes!

The next thing we need to do is by making use of the `AdapterServiceFactory`. This is a ServiceManager entry that will
look like the following:


```php
<?php
// Filename: /module/Album/config/module.config.php
return array(
    'db' => array(
        'driver'         => 'Pdo',
        'username'       => 'root',
        'password'       => 'admin',
        'dsn'            => 'mysql:dbname=album;host=localhost',
        'driver_options' => array(
            \PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES \'UTF8\''
        )
    ),
    'service_manager' => array(
        'factories' => array(
            'Album\Service\AlbumService' => 'Album\Service\Factory\AlbumServiceFactory',
            'Zend\Db\Adapter\Adapter'    => 'Zend\Db\Adapter\AdapterServiceFactory'
        )
    ),
    'view_manager'    => array(/** ... */),
    'controllers'     => array(/** ... */),
    'router'          => array(/** ... */)
);
```

Note the new Service that we called `Zend\Db\Adapter\Adapter`. Calling this Service will now always give back a
running instance of the `Zend\Db\Adapter\AdapterInterface` depending on what driver we assign. While you can give it a
completely different name like `dbadapter` but in ZF2 circles naming it `Zend\Db\Adapter\Adapter` is quite a standard.

With the adapter in place we're now able to run queries against the database. The construction of queries is best done
through the "QueryBuilder" features of `Zend\Db\Sql` which are `Zend\Db\Sql\Sql` for select queries,
`Zend\Db\Sql\Insert` for insert queries, `Zend\Db\Sql\Update` for update queries and `Zend\Db\Sql\Delete` for delete
queries. The basic workflow of these components is:

1. Build a query using `Sql`, `Insert`, `Update `or `Delete`
2. Create an Sql-Statement from the `Sql` object
3. Execute the query
4. Do something with the result

Knowing this we can now write the implementation for the `AlbumMapperInterface`.


Writing the mapper implementation
=================================

Our mapper implementation will reside inside the same namespace as its interface. Go ahead and create a class called
`ZendDbSqlMapper` and implement the `AlbumMapperInterface`.

```php
<?php
// Filename: /module/Album/src/Album/Mapper/ZendDbSqlMapper.php
namespace Album\Mapper;

class ZendDbSqlMapper implements AlbumMapperInterface
{
    /**
     * @param int|string $id
     *
     * @return \Album\Entity\AlbumInterface
     * @throws \Exception
     */
    public function find($id)
    {
    }

    /**
     * @return array|\Album\Entity\AlbumInterface[]
     */
    public function findAll()
    {
    }
}
```

Now recall what we have learned earlier. For `Zend\Db\Sql` to function we will need a working implementation of the
`AdapterInterface`. This is a requirement and therefore will be injected using constructor-injection. Create a
`__construct()` function that accepts an `AdapterInterface` as parameter and store it within the class.

```php
<?php
// Filename: /module/Album/src/Album/Mapper/ZendDbSqlMapper.php
namespace Album\Mapper;

use Zend\Db\Adapter\AdapterInterface;

class ZendDbSqlMapper implements AlbumMapperInterface
{
    /**
     * @var \Zend\Db\Adapter\AdapterInterface
     */
    protected $dbAdapter;

    /**
     * @param AdapterInterface  $dbAdapter
     */
    public function __construct(AdapterInterface $dbAdapter)
    {
        $this->dbAdapter = $dbAdapter;
    }

    /**
     * @param int|string $id
     *
     * @return \Album\Entity\AlbumInterface
     * @throws \Exception
     */
    public function find($id)
    {
    }

    /**
     * @return array|\Album\Entity\AlbumInterface[]
     */
    public function findAll()
    {
    }
}
```

With the adapter in place, let's now implement the function `findAll()`. All you need for this is to create a new `Sql`
object and call the selector against our database table `album`. You then get the statement from the `Sql` instance and
execute it.


```php
<?php
// Filename: /module/Album/src/Album/Mapper/ZendDbSqlMapper.php
namespace Album\Mapper;

use Zend\Db\Adapter\AdapterInterface;

class ZendDbSqlMapper implements AlbumMapperInterface
{
    /**
     * @var \Zend\Db\Adapter\AdapterInterface
     */
    protected $dbAdapter;

    /**
     * @param AdapterInterface  $dbAdapter
     */
    public function __construct(AdapterInterface $dbAdapter)
    {
        $this->dbAdapter = $dbAdapter;
    }

    /**
     * @param int|string $id
     *
     * @return \Album\Entity\AlbumInterface
     * @throws \Exception
     */
    public function find($id)
    {
    }

    /**
     * @return array|\Album\Entity\AlbumInterface[]
     */
    public function findAll()
    {
        $sql    = new Sql($this->dbAdapter);
        $select = $sql->select('album');

        $stmt   = $sql->prepareStatementForSqlObject($select);
        $result = $stmt->execute();

        \Zend\Debug\Debug::dump($result);die();
    }
}
```

The above code should look fairly straight forward to you. We do a dump from the `$result` variable to see what we get
here. If you go to your album index you'll see the following output:

```text
object(Zend\Db\Adapter\Driver\Pdo\Result)#303 (8) {
  ["statementMode":protected] => string(7) "forward"
  ["resource":protected] => object(PDOStatement)#296 (1) {
    ["queryString"] => string(29) "SELECT `album`.* FROM `album`"
  }
  ["options":protected] => NULL
  ["currentComplete":protected] => bool(false)
  ["currentData":protected] => NULL
  ["position":protected] => int(-1)
  ["generatedValue":protected] => string(1) "0"
  ["rowCount":protected] => NULL
}
```

As you can see we do not get any data returned. Instead we are presented with a dump of some `Result` object that
appears to have no data in it whatsoever. But this is a faulty assumption. This `Result` object only has information
available for you when you actually try to access it. To make use of the data within the `Result` object the best
approach would be to pass the `Result` object over into a `ResultSet` object, as long as the query was successful.

```php
<?php
// Filename: /module/Album/src/Album/Mapper/ZendDbSqlMapper.php
namespace Album\Mapper;

use Zend\Db\Adapter\AdapterInterface;
use Zend\Db\Adapter\Driver\ResultInterface;
use Zend\Db\ResultSet\ResultSet;

class ZendDbSqlMapper implements AlbumMapperInterface
{
    /**
     * @var \Zend\Db\Adapter\AdapterInterface
     */
    protected $dbAdapter;

    /**
     * @param AdapterInterface  $dbAdapter
     */
    public function __construct(AdapterInterface $dbAdapter)
    {
        $this->dbAdapter = $dbAdapter;
    }

    /**
     * @param int|string $id
     *
     * @return \Album\Entity\AlbumInterface
     * @throws \Exception
     */
    public function find($id)
    {
    }

    /**
     * @return array|\Album\Entity\AlbumInterface[]
     */
    public function findAll()
    {
        $sql    = new Sql($this->dbAdapter);
        $select = $sql->select('album');

        $stmt   = $sql->prepareStatementForSqlObject($select);
        $result = $stmt->execute();

        if ($result instanceof ResultInterface && $result->isQueryResult()) {
            $resultSet = new ResultSet();

            \Zend\Debug\Debug::dump($resultSet->initialize($result));die();
        }

        die("no data");
    }
}
```

Refreshing the page you should now see the dump of a `ResultSet` object that has a protected property `count` with a
value of `5`.

```text
object(Zend\Db\ResultSet\ResultSet)#304 (8) {
  ["allowedReturnTypes":protected] => array(2) {
    [0] => string(11) "arrayobject"
    [1] => string(5) "array"
  }
  ["arrayObjectPrototype":protected] => object(ArrayObject)#305 (1) {
    ["storage":"ArrayObject":private] => array(0) {
    }
  }
  ["returnType":protected] => string(11) "arrayobject"
  ["buffer":protected] => NULL
  ["count":protected] => int(2)
  ["dataSource":protected] => object(Zend\Db\Adapter\Driver\Pdo\Result)#303 (8) {
    ["statementMode":protected] => string(7) "forward"
    ["resource":protected] => object(PDOStatement)#296 (1) {
      ["queryString"] => string(29) "SELECT `album`.* FROM `album`"
    }
    ["options":protected] => NULL
    ["currentComplete":protected] => bool(false)
    ["currentData":protected] => NULL
    ["position":protected] => int(-1)
    ["generatedValue":protected] => string(1) "0"
    ["rowCount":protected] => int(2)
  }
  ["fieldCount":protected] => int(3)
  ["position":protected] => int(0)
}
```

Another very interesting property is `["returnType":protected] => string(11) "arrayobject"`. This tells us that all
database entries will be returned as an `ArrayObject`. And this is a little problem as the `AlbumMapperInterface`
requires us to return an array of `AlbumInterface` objects. Luckily there is a very simple option for us available to
make this happen. In the examples above we have used the default `ResultSet` object. There is also a
`HydratingResultSet` which will hydrate the given data into a provided object.

This means: if we tell the `HydratingResultSet` to use the database data to create `Album` objects for us, then it will
do exactly this. Let's modify our code:

```php
<?php
// Filename: /module/Album/src/Album/Mapper/ZendDbSqlMapper.php
namespace Album\Mapper;

use Zend\Db\Adapter\AdapterInterface;
use Zend\Db\Adapter\Driver\ResultInterface;
use Zend\Db\ResultSet\HydratingResultSet;

class ZendDbSqlMapper implements AlbumMapperInterface
{
    /**
     * @var \Zend\Db\Adapter\AdapterInterface
     */
    protected $dbAdapter;

    /**
     * @param AdapterInterface  $dbAdapter
     */
    public function __construct(AdapterInterface $dbAdapter)
    {
        $this->dbAdapter = $dbAdapter;
    }

    /**
     * @param int|string $id
     *
     * @return \Album\Entity\AlbumInterface
     * @throws \Exception
     */
    public function find($id)
    {
    }

    /**
     * @return array|\Album\Entity\AlbumInterface[]
     */
    public function findAll()
    {
        $sql    = new Sql($this->dbAdapter);
        $select = $sql->select('album');

        $stmt   = $sql->prepareStatementForSqlObject($select);
        $result = $stmt->execute();

        if ($result instanceof ResultInterface && $result->isQueryResult()) {
            $resultSet = new HydratingResultSet(new \Zend\Stdlib\Hydrator\ClassMethods(), new \Album\Model\Album());

            \Zend\Debug\Debug::dump($resultSet->initialize($result));die();
        }

        die("no data");
    }
}
```