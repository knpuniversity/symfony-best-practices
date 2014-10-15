Fast and Simple with the @Route and @ParamConverter Annotations
===============================================================

It's already common to see annotations used for Doctrine entities and validation.
But honestly, most projects still under-use them. And that's a shame, because
we can build things faster and with clearer code if we use them.

Setting up the Project
----------------------

So first, let's build a page with as little work as possible. Our project
is empty, except for an AppBundle that has a blog Post entity and some data
fixtures. Make sure to install the composer dependencies and initialize the
database:

.. code-block:: bash

    composer install
    php app/console doctrine:database:create
    php app/console doctrine:schema:create
    php app/console doctrine:fixtures:load

Let's also boot up the built-in web server and try things out by going to
``http://localhost:8000``:

.. code-block:: bash

    php app/console server:run

Annotation Routing
------------------

We'll start by building a page that lists all blog posts. *Normally* we'd
create a new route in a YAML file. Instead open up ``app/config/routing.yml``
and add a little import entry:

.. code-block:: yaml

    # app/config/routing.yml
    app_bundle_annotations:
        resource: "@AppBundle/Controller"
        type: annotation

This is a one-time thing. And now, Symfony will read annotation routes from
any file in the ``Controller/`` directory. Let's create that directory and
put a ``PostController`` class in it::

    // src/AppBundle/Controller/PostController.php
    namespace AppBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class PostController extends Controller
    {

    }

PHPStorm has a shortcut for this if you're using the Symfony2 plugin, but
you can also just create it by hand. There's a ``generate:controller`` console
command too, but right now, it needs some improvements.

Add an ``indexAction`` method and put an ``@Route`` annotation above it::

    // src/AppBundle/Controller/PostController.php
    // ...

    /**
     * @Route("/posts")
     */
    public function indexAction()
    {
        die('it works!');
    }

If we check to see if the route exists now, we'll get a huge error:

.. code-block:: bash

    php app/console router:debug

.. code-block:: text

    [Semantical Error] The annotation "@Route" in method
    AppBundle\Controller\PostController::indexAction() was never
    imported. Did you maybe forget to add a "use" statement for
    this annotation?

This feature comes from the `SensioFrameworkExtraBundle`_ and its docs have
the ``Route`` import line we need to add::

    // src/AppBundle/Controller/PostController.php
    // ...
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

    class PostController extends Controller
    {
        /**
         * @Route("/posts")
         */
        public function indexAction()
        {
            die('it works!');
        }
    }

It should work now, so let's go directly to ``http://localhost:8000/posts``
in your browser.

Rendering the Template
----------------------

Let's get this page finished quickly because I want to show you a few Symfony
Easter Eggs. Query for all the posts and pass them to a template, but *don't*
add any colons to the name. With no colons, Symfony just looks in
``app/Resources/views``. That's Easter Egg #1: you can stop using the weird
colons *and* keep all your templates in the same place::

    // src/AppBundle/Controller/PostController.php
    // ...

    /**
     * @Route("/posts")
     */
    public function indexAction()
    {
        $posts = $this->getDoctrine()
            ->getRepository('AppBundle:Post')
            ->findAll();

        return $this->render('Post/index.html.twig', array(
            'posts' => $posts,
        ));
    }

Use your mad-styling skills in the template to loop over the posts and print
them out:

.. code-block:: html+jinja

    {# app/Resources/views/Post/index.html.twig #}
    {% extends 'base.html.twig' %}

    {% block body %}
    <h1>POSTS!</h1>

    <ul>
        {% for post in posts %}
            <li>
                {{ post.title }}
            </li>
        {% endfor %}
    </ul>
    {% endblock %}

I'm refreshing to prove the I'm not lying.

Page 2 and the ParamConverter
-----------------------------

Let's see how fast we can create a page to show *one* Post. I'm adding a ``showAction``
and an annotation route with a ``/posts/{id}`` path::

    // src/AppBundle/Controller/PostController.php
    // ...

    /**
     * @Route("/posts/{id}")
     */
    public function showAction($id)
    {
        die('Mr Testers');
    }

That's enough to get the page working. Instead of having an ``$id`` argument
and querying for the Post, we can name the argument``$post`` and type-hint
it with the ``Post`` class::

    // src/AppBundle/Controller/PostController.php
    use AppBundle\Entity\Post;
    // ...

    /**
     * @Route("/posts/{id}")
     */
    public function showAction(Post $post)
    {
        die('Mr Testers');
    }

That's Easter Egg #2: if you type-hint a controller argument, Symfony will
query for that object using the wildcards in the route.

Create the template and render a few things::

    // src/AppBundle/Controller/PostController.php
    // ...

    /**
     * @Route("/posts/{id}")
     */
    public function showAction(Post $post)
    {
        return $this->render('Post/show.html.twig', array(
            'post' => $post
        ));
    }

.. code-block:: html+jinja

    {# app/Resources/views/Post/show.html.twig #}
    {% extends 'base.html.twig' %}

    {% block body %}
    <h1>{{ post.title }}</h1>

    <div>
        {{ post.contents }}
    </div>
    {% endblock %}

And now let's try it! We only touched 2 files to create this page and didn't
even need to make a query directly. The easter egg I just showed you is called
the ``ParamConverter`` and comes from that same `SensioFrameworkExtraBundle`_.
As long as the routing wildcard matches a property on your entity, it works!
You can theoretically configure it to be smarter, but since the syntax is
ugly, I'd rather just query manually if it doesn't work.

Route Name
----------

To link the pages together, the route needs a name, so let's give it the name post_show::

    // src/AppBundle/Controller/PostController.php
    // ...

    /**
     * @Route("/posts/{id}", name="post_show")
     */
    public function showAction(Post $post)
    {
        return $this->render('Post/show.html.twig', array(
            'post' => $post
        ));
    }

Now that it has a name, we can create links using the good ol' ``path()``
function:

    <ul>
        {% for post in posts %}
            <li>
                <a href="{{ path('post_show', { 'id': post.id }) }}">
                    {{ post.title }}
                </a>
            </li>
        {% endfor %}
    </ul>
    {% endblock %}

I don't use *all* of Symfony's annotations, but I do like the ones that let
me create *less* files. Here we made 2 pages with just 3 files, which I think
is pretty great.

.. _`SensioFrameworkExtraBundle`:
