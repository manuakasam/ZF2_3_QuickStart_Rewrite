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

- Blabla, getting data from DB is best done using a ServiceLayer
- Modifying Album\Controller\List with PseudoCode (pseudo as in, the final code that has no effect yet)
