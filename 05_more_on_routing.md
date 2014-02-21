Understanding the Router
========================

Right now we have a pretty solid set up for our module. However we're not really doing all too much yet, to be
precise, all we do is display all `Album` entries on one page. In this chapter you will learn everything you need
to know about the `Router` to create other routes to be able to display only a single album, to add new albums
to your application and to edit and delete existing albums.


Different route types
=====================

Before we go into details on our application, let's take a look at the most important route types that Zend
Framework offers.

**`Zend\Mvc\Router\Http\Literal`**
----------------------------------

The first common route type is the `Literal`-Route. As mentioned in a previous chapter a literal route is one that
matches a specific string. Examples for URLs that are usually literal routes are:

- http://domain.com/album
- http://domain.com/album/add
- http://domain.com/about-me
- http://domain.com/my/very/deep/page
- http://domain.com/my/very/deep/page

Configuration for a literal route requires you to set up the route that should be matched and needs you to define
some defaults to be used, for example which controller and which action to call. A simple configuration for a
literal route looks like this:

```php
'router' => array(
    'routes' => array(
        'about' => array(
            'type' => 'Zend\Mvc\Router\Http\Literal',
            'options' => array(
                'route'    => '/about-me',
                'defaults' => array(
                    'controller' => 'AboutMeController',
                    'action'     => 'aboutme',
                ),
            ),
        )
    )
)
```

**`Zend\Mvc\Router\Http\Segment`**
----------------------------------

The second most commonly used route type is the `Segment`-Route. A segmented route is used for whenever your url
is supposed to contain variable parameters. Pretty often those parameters are used to identify certain objects
within your application. Some examples for URLs that contain parameters and are usually segment routes are:

- http://domain.com/album/1                     // parameter "1"
- http://domain.com/album/details/1             // parameter "1"
- http://domain.com/album/edit/1                // parameter "1"
- http://domain.com/album/1/edit                // parameter "1"
- http://domain.com/news/archive/2014           // parameter "2014"
- http://domain.com/news/archive/2014/january   // parameter "2014" and "january"

Configuring a `Segment`-Route takes a little more effort but isn't difficult to understand. The tasks you have to
do are similar at first, you have to define the route-type, just be sure to make it `Segment`. Then you have to
define the route and add parameters to it. Then as usual you define the defaults to be used, the only thing that
differs in this part is that you can assign defaults for your parameters, too. The new part that is used on routes
of the `Segment` type is to define so called `constraints`. They are used to tell the `Router` what "rules" are
given for parameters. For example, an `id`-parameter is only allowed to be of type `integer`, the `year`-parameter
is only allowed to be of type `integer` and may only contain exactly `four digits`. A sample configuration can
look like this:

```php
'router' => array(
    'routes' => array(
        'archives' => array(
            'type' => 'Zend\Mvc\Router\Http\Segment',
            'options' => array(
                'route'    => '/news/archive/:year',
                'defaults' => array(
                    'controller' => 'ArchiveController',
                    'action'     => 'byYear',
                ),
                'constraints' => array(
                    'year' => '\d{4}'
                )
            ),
        )
    )
)
```

This configuration defines a route for an URL like `domain.com/news/archive/2014`. As you can see we our route now
contains the part `:year`. This is called a route-parameter. Route parameters for `Segment`-Routes are defined by a
in front of a string. The string then is the `name` of the parameter.

Under `constraints` you see that we have another array. This array contains regular expression rules for each
parameter of your route. In our example case the regex uses two parts, the first one being `\d` which means "a
digit", so any number from 0-9. The second part is `{4}` which means that the part before this has to match exactly
four times. So in easy words we say "four digits".

If now you call the URL `domain.com/news/archive/123`, the router will not match the URL because we only support
years with four digits.

You may notice that we did not define any `defaults` for the parameter `year`. This is because the parameter is
currently set up as a `required` parameter. If a parameter is supposed to be `optional` we need to define this
inside the route definition. This is done by adding square brackets around the parameter. Let's modify the above
example route to have the `year` parameter optional and use the current year as default:

```php
'router' => array(
    'routes' => array(
        'archives' => array(
            'type' => 'Zend\Mvc\Router\Http\Segment',
            'options' => array(
                'route'    => '/news/archive[/:year]',
                'defaults' => array(
                    'controller' => 'ArchiveController',
                    'action'     => 'byYear',
                    'year'       => date('Y')
                ),
                'constraints' => array(
                    'year' => '\d{4}'
                )
            ),
        )
    )
)
```

Notice that now we have a part in our route that is optional. Not only the parameter `year` is optional. The slash
that is separating the `year` parameter from the URL string `archive` is optional, too, and may only be there
whenever the `year` parameter is present.


Different routing concepts
==========================

When thinking about the whole application it becomes clear that there are a lot of routes to be matched. When
writing these routes you have two options to decide from. One option to spent less time writing routes but in turn
are a little bit slow in matching. And another option to write very explicit routes that match a little bit faster
but require more work to define them. Let's take a loot at both of them.

General routes
--------------

A general route is one that matches many URLs. You may remember this concept from Zend Framework 1 where basically
you didn't even bother about routes because we had one "god route" that was used for everything. You define the
controller, the action, and all parameters within just one single route.

The big advantage of this approach is the immense time you save when developing your application. The downside
however is that matching such a route can take a little bit longer due to the fact that so many variables need to
be checked. However, as long as you not overdo it, this is a viable concept. For this reason the
ZendSkeletonApplication uses a very general route, too. Let's take a look at a general route:

```php
'router' => array(
    'routes' => array(
        'default' => array(
            'type' => 'Zend\Mvc\Router\Http\Segment',
            'options' => array(
                'route'    => '/[:controller[/:action]]',
                'defaults' => array(
                    '__NAMESPACE__' => 'Application\Controller',
                    'controller'    => 'Index',
                    'action'        => 'index',
                ),
                'constraints' => [
                    'controller' => '[a-zA-Z][a-zA-Z0-9_-]*',
                    'action'     => '[a-zA-Z][a-zA-Z0-9_-]*',
                ]
            ),
        )
    )
)
```

Let's take a closer look as to what has been defined in this configuration. The `route` part now contains two
optional parameters `controller` and `action`. The `action` parameter is optional only when the `controller`
parameter is present.

Within the `defaults`-section it looks a little bit different, too. The `__NAMESPACE__` will be used to concatenate
with the `controller` parameter at all times. So for example when the `controller` parameter is "news" then the
`controller` to be called from the `Router` will be `Application\Controller\news`, if the parameter is "archive"
the `Router` will call the controller `Application\Controller\archive`.

The `defaults`-section then is pretty straight forward again. Both parameters, `controller` and `action`, only
have to follow the conventions given by PHP-Standards. They have to start with a letter from `a-z`, upper- or
lowercase and after that first letter there can be an (almost) infinite amount of letters, digits, underscores or
dashes.

**The big downside** to this approach not only is that matching this route is a little slower, it is that there
is no error-checking going on. For example, when you were to call an URL like `domain.com/weird/doesntExist` then
the `controller` would be "Application\Controller\weird" and the `action` would be "doesntExistAction". As you can
guess by the names let's assume neither `controller` nor `action` does exist. The route will still match but an
`Exception` will be thrown because the `Router` will be unable to find the requested resources and we'll receive
a `404`-Response.