Making use of Forms and Fieldsets
=================================

So far all we did was read data from the database. In a real-life-application this won't get us very far as very often
the least we need to do is to support full `Create`, `Read`, `Update` and `Delete` operations (CRUD). Most often the
process of getting data into our database is that a user enters the data into a web `<form>` and the application then
uses the user input saves it into our backend.

We want to be able to do exactly this and Zend Framework provides us with all the tools we need to achieve our goal.
Before we jump into coding, we need to understand the two core components for this task first. So let's take a look at
what these components are and what they are used for.

Zend\Form\Fieldset
------------------

The first component that you have to know about is `Zend\Form\Fieldset`. A `Fieldset` is a component that contains a
reusable set of elements. You will use the `Fieldset` to create the frontend-input for your backend-models. It is
considered best practice to have one `Fieldset` for every `Model` of your application.

The `Fieldset`-component however is no `Form`, meaning you will not be able to use a `Fieldset` without attaching it
to the `Form`-component. The advantage here is that you have one set of elements that you can re-use for as many
`Forms` as you like without having to re-declare all the inputs for the `Model` that's representet by the `Fieldset`.

Zend\Form\Form
--------------

The main component you'll need and that most probably you've heard about already is `Zend\Form\Form`. The `Form`-
component is the main container for all elements of your web `<form>`. You are able to add single elements or a set of
elements in the form of a `Fieldset`, too.


Creating your first Fieldset
============================

Explaining how the `Zend\Form` component works is best done by giving you real code to work with. So let's jump right
into it and create all the forms we need to finish our `Album` module. We start by creating a `Fieldset` that contains
all the input elements that we need to work with our `Album`-data.

- You will need one hidden input for the `id` property, which is only needed for editting and deleting data.
- You will need one text input for the `artist` property
- You will need one text input for the `title` property

Create the file `/module/Album/src/Album/Form/AlbumFieldset.php` and add the following code:

```php
<?php
// Filename: /module/Album/src/Album/Form/AlbumFieldset.php
namespace Album\Form;

use Zend\Form\Fieldset;

class AlbumFieldset extends Fieldset
{
    public function __construct()
    {
        $this->add(array(
            'type' => 'hidden',
            'name' => 'id'
        ));

        $this->add(array(
            'type' => 'text',
            'name' => 'artist',
            'options' => array(
                'label' => 'The Artist'
            )
        ));

        $this->add(array(
            'type' => 'text',
            'name' => 'title',
            'options' => array(
                'label' => 'Album Title'
            )
        ));
    }

}
```

As you can see this class is pretty handy. All we do is to have our class extend `Zend\Form\Fieldset` and then we
overwrite the parents `__construct()` method and add all the elements we need to the fieldset. This `Fieldset` can now
be used by as many forms as we want. So let's go ahead and create our first `Form`.


Creating the InsertAlbumForm
============================

Now that we have our `AlbumFieldset` in place, we need to use it inside a `Form`. We then need to add a Submit-Button
to the form so that the user will be able to submit the data and we're done. So create the `InsertAlbumForm` within the
same directory under `/module/Album/src/Album/Form/InsertAlbumForm` and add the `AlbumFieldset` to it:


```php
<?php
// Filename: /module/Album/src/Album/Form/InsertAlbumForm.php
namespace Album\Form;

use Zend\Form\Form;

class InsertAlbumForm extends Form
{
    public function __construct()
    {
        $this->add(array(
            'type' => 'Album\Form\AlbumFieldset'
        ));

        $this->add(array(
            'type' => 'submit',
            'name' => 'submit',
            'attributes' => array(
                'value' => 'Insert new Album'
            )
        ));
    }
}
```

And that's our form. Nothing special here, we add our `AlbumFieldset` to the Form, we add a submit button to the form
and nothing more. Let's now make use of the Form.


Adding a new Album
==================

Now that we have the `InsertAlbumForm` written we want to use it. But there's a couple more tasks that you need to do.
The tasks that are standing right in front of you are:

- create a new controller `InsertController`
- add `AlbumService` as a dependency to the `InsertController`
- add `InsertAlbumForm` as a dependency to the `InsertController`
- create a new route `album/insert` that routes to the `InsertController`
- create a new view that displays the form


Creating the `InsertController`
-------------------------------

As you can see from the task-list we need a new controller and this controller is supposed to have two dependencies.
One dependency being the `AlbumService` that's also being used within our `ListController` and the other dependency
being the `InsertAlbumForm` which is new. Since the `InsertAlbumForm` is a dependency that the `ListController` doesn't
need to display album-data, we will create a new controller to keep things properly separated. First, register a
controller-factory within the configuration:

```php
<?php
// Filename: /module/Album/config/module.config.php
return array(
    'db' => array(),
    'service_manager' => array(),
    'view_manager' => array(),
    'controllers' => array(
        'factories' => array(
            'Album\Controller\List'   => 'Album\Controller\Factory\ListControllerFactory',
            'Album\Controller\Insert' => 'Album\Controller\Factory\InsertControllerFactory'
        )
    ),
    'router' => array()
);
```

Nest step would be to write the `InsertControllerFactory`. Have the factory return the `InsertController` and add the
required dependencies within the constructor.

```php
<?php
// Filename: /module/Album/src/Album/Controller/Factory/InsertControllerFactory.php
namespace Album\Controller\Factory;

use Album\Controller\InsertController;
use Zend\ServiceManager\FactoryInterface;
use Zend\ServiceManager\ServiceLocatorInterface;

class InsertControllerFactory implements FactoryInterface
{
    public function createService(ServiceLocatorInterface $serviceLocator)
    {
        $realServiceLocator = $serviceLocator->getServiceLocator();
        $albumService       = $realServiceLocator->get('Album\Service\AlbumService');
        $albumInsertForm    = $realServiceLocator->get('FormElementManager')->get('Album\Form\InsertAlbumForm');

        return new InsertController(
            $albumService,
            $albumInsertForm
        );
    }

}
```

Next up is the creation of our controller. Be sure to type hint the dependencies by their interfaces!

```php
<?php
// Filename: /module/Album/src/Album/Controller/InsertController.php
namespace Album\Controller;

use Album\Service\AlbumServiceInterface;
use Zend\Form\FormInterface;
use Zend\Mvc\Controller\AbstractActionController;

class InsertController extends AbstractActionController
{
    protected $albumService;

    protected $albumForm;

    public function __construct(
        AlbumServiceInterface $albumService,
        FormInterface $albumForm
    ) {
        $this->albumService = $albumService;
        $this->albumForm    = $albumForm;
    }
}
```

Right on to creating the new route:

```php
<?php
// Filename: /module/Album/config/module.config.php
return array(
    'db'              => array( /** ... */ ),
    'service_manager' => array( /** ... */ ),
    'view_manager'    => array( /** ... */ ),
    'controllers'     => array( /** ... */ ),
    'router' => array(
        'routes' => array(
            'album' => array(
                'type' => 'literal',
                'options' => array(
                    'route'    => '/album',
                    'defaults' => array(
                        'controller' => 'Album\Controller\List',
                        'action'     => 'index',
                    ),
                ),
                'may_terminate' => true,
                'child_routes'  => array(
                    'detail' => array(
                        'type' => 'segment',
                        'options' => array(
                            'route'    => '/:id',
                            'defaults' => array(
                                'action' => 'detail'
                            ),
                            'constraints' => array(
                                'id' => '[1-9]\d*'
                            )
                        )
                    ),
                    'insert' => array(
                        'type' => 'literal',
                        'options' => array(
                            'route'    => '/insert',
                            'defaults' => array(
                                'controller' => 'Album\Controller\Insert'
                            )
                        )
                    )
                )
            )
        )
    )
);
```

And lastly let's create a dummy template:

```hmtl
<!-- Filename: /module/Album/view/album/insert/index.phtml -->
<h1>InsertController::indexAction()</h1>
```

**Checking the current status**

If you try to access the new route `domain.loc/album/insert` you're supposed to see the following error message:

```text
Fatal error: Call to a member function insert() on a non-object in
{libraryPath}/Zend/Form/Fieldset.php on line {lineNumber}
```

If this is not the case, be sure to follow the tutorial correctly and carefully check all your files. Assuming you are
getting this error, let's find out what it means and fix it!


The above error message is very common and it's solution isn't that intuitive. It appears that there is an error within
the `Zend/Form/Fieldset.php` but that's not the case. The error message let's you know that something didn't go right
while you were creating your form. In fact, while creating both the `InsertAlbumForm` as well as the `AlbumFieldset` we
have forgotten something very, very important.

**When overwriting a `__construct()` method within the `Zend\Form`-component, be sure to always call
`parent::__construct()`!** Without this, forms and fieldsets will not be able to get initiated correctly. Let's now fix
the problem by calling the parents constructor in both form and fieldset.

First the form
```php
<?php
// Filename: /module/Album/src/Album/Form/InsertAlbumForm.php
namespace Album\Form;

use Zend\Form\Form;

class InsertAlbumForm extends Form
{
    public function __construct()
    {
        parent::__construct('insert-album-form');

        $this->add(array(
            'type' => 'Album\Form\AlbumFieldset'
        ));

        $this->add(array(
            'type' => 'submit',
            'name' => 'submit',
            'attributes' => array(
                'value' => 'Insert new Album'
            )
        ));
    }
}
```

And then the fieldset
```php
<?php
// Filename: /module/Album/src/Album/Form/AlbumFieldset.php
namespace Album\Form;

use Zend\Form\Fieldset;

class AlbumFieldset extends Fieldset
{
    public function __construct()
    {
        parent::__construct('album-fieldset');

        $this->add(array(
            'type' => 'hidden',
            'name' => 'id'
        ));

        $this->add(array(
            'type' => 'text',
            'name' => 'artist',
            'options' => array(
                'label' => 'The Artist'
            )
        ));

        $this->add(array(
            'type' => 'text',
            'name' => 'title',
            'options' => array(
                'label' => 'Album Title'
            )
        ));
    }

}
```

Reloading your website now will yield you the (current) desired result. However we're making use of a Feature from the
`FormElementManager` or rather it's parent the `AbstractPluginManager` which is called `autoAddInvokableClass`. This
feature tries to access given classnames even when we have not registered them within the ServiceManager configuration.

While this does work in our case it is considered a bad-practice to make use of it. So let's manually assign the
invokables for our form and the fieldset within the configuration for the `FormElementManager`!

```php
<?php
// Filename: /module/Album/config/module.config.php
return array(
    'form_elements' => array(
        'invokables' => array(
            'Album\Form\AlbumFieldset'   => 'Album\Form\AlbumFieldset',
            'Album\Form\InsertAlbumForm' => 'Album\Form\InsertAlbumForm'
        )
    ),
    'db'              => array( /** ... */ ),
    'service_manager' => array( /** ... */ ),
    'view_manager'    => array( /** ... */ ),
    'controllers'     => array( /** ... */ ),
    'router'          => array( /** ... */ )
);
```


Displaying the form
===================

Now that we have our `InsertAlbumForm` within our `InsertController` it's time to pass this form to the view and have
it rendered using the provided `ViewHelpers` from the `Zend\Form` component. First change your controller so that the form
is passed to the view.

```php
<?php
// Filename: /module/Album/src/Album/Controller/InsertController.php
namespace Album\Controller;

use Album\Service\AlbumServiceInterface;
use Zend\Form\FormInterface;
use Zend\Mvc\Controller\AbstractActionController;
use Zend\View\Model\ViewModel;

class InsertController extends AbstractActionController
{
    protected $albumService;

    protected $albumForm;

    public function __construct(
        AlbumServiceInterface $albumService,
        FormInterface $albumForm
    ) {
        $this->albumService = $albumService;
        $this->albumForm    = $albumForm;
    }

    public function indexAction()
    {
        return new ViewModel(array(
            'form' => $this->albumForm
        ));
    }
}
```

And then we need to modify our view to have the form rendered.


```php
<!-- Filename: /module/Album/view/album/insert/index.phtml -->
<h1>InsertController::indexAction()</h1>
<?php
$form = $this->form;
$form->setAttribute('method', 'POST');
$form->setAttribute('action', $this->url());
$form->prepare();

echo $this->form()->openTag($form);

echo $this->formCollection($form);

echo $this->form()->closeTag();
```

We do a couple of different things in here. Firstly, we tell the form that it is supposed to use the `POST` method
rather than `GET`. Then we tell the form that it should send it's data to the current URL and then we tell the form to
`prepare()` itself which triggers a couple of internal things.

Next we're using a couple of `ViewHelpers` which take care of rendering the form for us. There's many different ways to
render a form within Zend Framework but using `formCollection()` is probably the fastest one. Please refer to the
appendix section "Rendering Zend\Form" for further information on how to render your forms differently. //@todo appendix link

Refreshing the browser you will now see your form properly displayed. However if we're submitting the form all we see
is our form being displayed again. And this is due to the simple fact that we didn't add any logic to the controller
yet.


Controller Logic for basically all Forms
========================================

Writing a Controller that handles a form workflow is pretty simple and it's basically identical for each and every
form you have within your application.

1. You want to check if the current request is a POST-Request, meaning if the form has been sent
2. If the form has been sent, you want to:
    - store the POST-Data within the Form
    - check if the form passes validation
3. If the form passes validation, you want to:
    - pass the form data to your service to have it stored
    - redirect the user to either the detail page of the entered data or to some overview page
4. In all other cases, you want the form displayed, sometimes alongside given error messages

And all of this is really not that much code. Modify your `InsertController` to the following code:

```php
<?php
// Filename: /module/Album/src/Album/Controller/InsertController.php
namespace Album\Controller;

use Album\Service\AlbumServiceInterface;
use Zend\Form\FormInterface;
use Zend\Mvc\Controller\AbstractActionController;
use Zend\View\Model\ViewModel;

class InsertController extends AbstractActionController
{
    protected $albumService;

    protected $albumForm;

    public function __construct(
        AlbumServiceInterface $albumService,
        FormInterface $albumForm
    ) {
        $this->albumService = $albumService;
        $this->albumForm    = $albumForm;
    }

    public function indexAction()
    {
        $request = $this->getRequest();

        if ($request->isPost()) {
            $this->albumForm->setData($request->getPost());

            if ($this->albumForm->isValid()) {
                try {
                    $this->albumService->saveAlbum($this->albumForm->getData());

                    return $this->redirect()->toRoute('album');
                } catch (\Exception $e) {
                    // Some DB Error happened, log it and let the user know
                }
            }
        }

        return new ViewModel(array(
            'form' => $this->albumForm
        ));
    }
}
```

This example code should be pretty straight forward. First we save the current request into a local variable. Then we
check if the current request ist a POST-Request and if so, we store the requests POST-data into the form. If the form
turns out to be valid we try to save the form data through our service and then redirect the user to the route `album`.
If any error occurred at any point we simply display the form again.

Submitting the form right now will return into the following error

```text
Fatal error: Call to undefined method Album\Service\AlbumService::save() in
/module/Album/src/Album/Controller/InsertController.php on line 33
```

Let's fix this by extending our `AlbumService`. Be sure to also change the signature of the `AlbumServiceInterface`!

```php
<?php
// Filename: /module/Album/src/Album/Service/AlbumServiceInterface.php
namespace Album\Service;

use Album\Model\AlbumInterface;

interface AlbumServiceInterface
{
    /**
     * Should return a set of all albums that we can iterate over. Single entries of the array or \Traversable object
     * should be of type \Album\Model\Album
     *
     * @return array|\Traversable|AlbumInterface[]
     */
    public function findAllAlbums();

    /**
     * Should return a single album
     *
     * @param  int $id Identifier of the Album that should be returned
     * @return AlbumInterface
     */
    public function findAlbum($id);

    /**
     * Should save a given implementation of the AlbumInterface and return it. If it is an existing Album the Album
     * should be updated, if it's a new Album it should be created.
     *
     * @param  AlbumInterface $album
     * @return AlbumInterface
     */
    public function saveAlbum(AlbumInterface $album);
}
```

We changed our interface slightly to typehint against the `AlbumInterface` rather than against it's implementation. The
`saveAlbum()` function has been added and needs to be implemented within the `AlbumService` now.

```php
<?php
// Filename: /module/Album/src/Album/Service/AlbumService.php
namespace Album\Service;

use Album\Model\AlbumInterface;
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
        $resultSet = $this->tableGateway->select();

        return $resultSet;
    }

    /**
     * @inheritDoc
     */
    public function findAlbum($id)
    {
        $id     = (int) $id;
        $rowset = $this->tableGateway->select(array(
            'id' => $id
        ));

        $row  = $rowset->current();

        if (!$row) {
            throw new \InvalidArgumentException("Could not find row $id");
        }

        return $row;
    }

    /**
     * @inheritDoc
     */
    public function saveAlbum(AlbumInterface $album)
    {
        if (null === $album->getId()) {
            $this->tableGateway->insert($album);
        } else {
            $this->tableGateway->update($album, array(
                'id' => $album->getId()
            ));
        }

        return $album;
    }
}
```

As you can see the `saveAlbum()` function checks if the current `Album` already has an `id` set. If it is not the case
we call the `insert()` function of the TableGateway but if it's the case we call the ``update() function and assign the
id. Lastly we return the album again.

Let's submit our form again and see what we get.

```text
Catchable fatal error: Argument 1 passed to Album\Service\AlbumService::saveAlbum()
must implement interface Album\Model\AlbumInterface, array given,
called in /module/Album/src/Album/Controller/InsertController.php on line 33
and defined in /module/Album/src/Album/Service/AlbumService.php on line 49
```

Forms, per default, give you data in an array format. But our `AlbumService` expects the format to be an implementation
of the `AlbumInterface`. This means we need to find a way to have this array data become object data. If you recall the
previous chapter, this is done through the use of hydrators.


Zend\Form and Zend\Stdlib\Hydrator working together
===================================================

As you recall from the previous chapter, the Zend\Db\TableGateway-component had the ability to make use of hydration
internally. Zend\Form does provide us with the same abilities like the TableGateway does! We do have the ability to
assign hydrators to both `Zend\Form\Form` and `Zend\Form\Fieldset`. This means the big question is where to add thy
hydrators to.

Now remember that we have defined a `Fieldset` that matches our `Album`-Model. No matter where we are going to use this
fieldset, we always want it to return an `Album`-Model. It wouldn't make sense to apply a hydrator to one form and then
basically copy/paste the same stuff to every other form that uses the `AlbumFieldset`. So the right choice would be to
add the hydration to the fieldset. Before we do this however, let's modify our controller to temporarily dump the data
that's coming from the form.

```php
<?php
// Filename: /module/Album/src/Album/Controller/InsertController.php
namespace Album\Controller;

use Album\Service\AlbumServiceInterface;
use Zend\Form\FormInterface;
use Zend\Mvc\Controller\AbstractActionController;
use Zend\View\Model\ViewModel;

class InsertController extends AbstractActionController
{
    protected $albumService;

    protected $albumForm;

    public function __construct(
        AlbumServiceInterface $albumService,
        FormInterface $albumForm
    ) {
        $this->albumService = $albumService;
        $this->albumForm    = $albumForm;
    }

    public function indexAction()
    {
        $request = $this->getRequest();

        if ($request->isPost()) {
            $this->albumForm->setData($request->getPost());

            if ($this->albumForm->isValid()) {
                try {
                    \Zend\Debug\Debug::dump($this->albumForm->getData());
                    die();

                    $this->albumService->saveAlbum($this->albumForm->getData());

                    return $this->redirect()->toRoute('album');
                } catch (\Exception $e) {
                    // Some DB Error happened, log it and let the user know
                }
            }
        }

        return new ViewModel(array(
            'form' => $this->albumForm
        ));
    }
}
```

Submit the form once with some dummy data and check the output. It should look something like this:

```text
array(2) {
    ["submit"] => string(16) "Insert new Album"
    ["album-fieldset"] => array(3) {
        ["id"] => string(0) ""
        ["artist"] => string(4) "test"
        ["title"] => string(3) "123"
    }
}
```

This is the data from the whole form, including the submit button. And you can see the album-data that resides within
the `album-fieldset` key, which is the name of the `AlbumFieldset` class. Let's now add hydration to the
`AlbumFieldset`.

```php
<?php
// Filename: /module/Album/src/Album/Form/AlbumFieldset.php
namespace Album\Form;

use Album\Model\Album;
use Zend\Form\Fieldset;
use Zend\Stdlib\Hydrator\ClassMethods;

class AlbumFieldset extends Fieldset
{
    public function __construct()
    {
        parent::__construct('album-fieldset');

        $this->setHydrator(new ClassMethods(false))
             ->setObject(new Album());

        $this->add(array(
            'type' => 'hidden',
            'name' => 'id'
        ));

        $this->add(array(
            'type' => 'text',
            'name' => 'artist',
            'options' => array(
                'label' => 'The Artist'
            )
        ));

        $this->add(array(
            'type' => 'text',
            'name' => 'title',
            'options' => array(
                'label' => 'Album Title'
            )
        ));
    }
}
```

As you can see we're doing two things. We tell the fieldset to be using the `ClassMethods` hydrator and then we tell
the fieldset that the default object to be returned is our `Album`-Model. However when you're re-submitting the form
now you'll notice that nothing has changed. We're still only getting array data returned and no object.

This is due to the fact that the form itself doesn't know that it has to return an object. When the form doesn't know
that it's supposed to return an object it uses the `ArraySeriazable` hydrator recursively. To change this, all we need
to do is to make our `AlbumFieldset` a so-called `base_fieldset`.

A `base_fieldset` basically tells the form "this form is all about me, don't worry about other data, just worry about
me". And when the form knows that this fieldset is the real deal, then the form will use the hydrator presented by the
fieldset and return the object that we desire. This is how it would look like:

```php
<?php
// Filename: /module/Album/src/Album/Form/InsertAlbumForm.php
namespace Album\Form;

use Zend\Form\Form;

class InsertAlbumForm extends Form
{
    public function __construct()
    {
        parent::__construct('insert-album-form');

        $this->add(array(
            'type' => 'Album\Form\AlbumFieldset',
            'options' => array(
                'use_as_base_fieldset' => true
            )
        ));

        $this->add(array(
            'type' => 'submit',
            'name' => 'submit',
            'attributes' => array(
                'value' => 'Insert new Album'
            )
        ));
    }
}
```

When adding the fieldset to the form all you have to do is set the options key `use_as_base_fieldset` to true and
you're done. Re-Submit the form now and you should see the following output:

```text
object(Album\Model\Album)#262 (3) {
    ["id":protected]     => string(0) ""
    ["title":protected]  => string(3) "best title"
    ["artist":protected] => string(3) "any artist"
}
```

Since we're getting an object returned from the form now, re-factor your controller and remove the data-dumping and
the new album can be saved to the database. //@todo that's utter bullshit :S i suck!

```php
<?php
// Filename: /module/Album/src/Album/Controller/InsertController.php
namespace Album\Controller;

use Album\Service\AlbumServiceInterface;
use Zend\Form\FormInterface;
use Zend\Mvc\Controller\AbstractActionController;
use Zend\View\Model\ViewModel;

class InsertController extends AbstractActionController
{
    protected $albumService;

    protected $albumForm;

    public function __construct(
        AlbumServiceInterface $albumService,
        FormInterface $albumForm
    ) {
        $this->albumService = $albumService;
        $this->albumForm    = $albumForm;
    }

    public function indexAction()
    {
        $request = $this->getRequest();

        if ($request->isPost()) {
            $this->albumForm->setData($request->getPost());

            if ($this->albumForm->isValid()) {
                try {
                    $this->albumService->saveAlbum($this->albumForm->getData());

                    return $this->redirect()->toRoute('album');
                } catch (\Exception $e) {
                    // Some DB Error happened, log it and let the user know
                }
            }
        }

        return new ViewModel(array(
            'form' => $this->albumForm
        ));
    }
}
```

