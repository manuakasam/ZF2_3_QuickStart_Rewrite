Getting Started
===============

**Environment**

- Suggested Tools
- Installing ZendSkeletonApplication
- Setting up a Virtual Host

**Explaining the Skeletton**

- explain ./config/application.config.php
- explain ./config/autoload/...
- explain ./data
- explain ./module
- explain ./public
- explain ./vendor

**Writing your First Module**

- Writing Module.php with nothing and making sure module able to be loaded
- Introducing Configuration, overwrite the default / route to our module (error message, we have no controller yet, no autoloaderconfig)
- Writing autoloader configuration, explaining StandardAutoloader, hinting ClassmapAutoloder(separate .md)
- Writing Album\Controller\ListController, only listAction (no list.phtml yet, explaining error message)
- Writing the view file, simple hello world, no data yet
- Future, getting data from DB

**Introducing the Service Layer**

- Blabla, what is a ServiceLayer
- Writing Album\Service\AlbumService with pseudo-data inline
- Dependency Injection, accessing AlbumService in ListController
- Hinting concrete differences between invokables and factories

**Introducing the DB Layer**

- Blabla, explaining the general workflow, DbAdapter Configuration, TableGateway, Table
- Writing the DbAdapter Configuration
- Writing AlbumTableGateway
- Writing AlbumTable
- Dependency Injection, Access to AlbumTable in AlbumService
- Modifying AlbumService, privide a findAll() function that accesses AlbumTable->fetchAll()
- Hinting to ListController which uses $albumService->findAll()

**Introducing the View Layer**

- Modifying ListController to return a ViewModel with ViewVariables
- writing Album\view\album\list\list.phtml
- hinting $this->foo equals $foo automatically
- talking about translations
- talking about escaping

**Adding new Albums**

- Introducing Zend\Form\Fieldset
- Writing Album\Form\AlbumFieldset
- Writing Album\Form\AddForm
- Writing Album\Controller\AddController
- Writing configuration
- Writing Album\view\album\add\add.phtml
- Example of Form-Output as array
- Problem: we want Album\Model\Album as FormOutput
- Introducing Zend\Stdlib\Hydrator (Zend\Hydrator ZF3)
- Modifying code, etc..

**Editting existing Albums**

- Writing a new Route
- Writing a new Controller
- Writing a new EditForm
- Explaining binding of objects to a Form

**Deleting existing Albums**

... too tired

