Create new controllers (routes)
--------------------------------

Newscoop plugins system is based on the Symfony Bundles system - so (almost) all Symfony features are avaiable.
If you want create own new controller (with routing) then you must only do a few things, first - create controller class:

.. code-block:: php

    // ExamplePluginBundle/Controller/LifecycleSubscriber.php
    <?php

    namespace Newscoop\ExamplePluginBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Symfony\Component\HttpFoundation\Request;

    class DefaultController extends Controller
    {
        /**
         * @Route("/testnewscoop")
         */
        public function indexAction(Request $request)
        {
            return $this->render('NewscoopExamplePluginBundle:Default:index.html.smarty');
        }
    }


In this example we will use annotations for configuration (``@Route("/testnewscoop")``).
Second we must register our Controller class (routing) in system:

.. code-block:: php

    // ExamplePluginBundle/Resources/config/routing.yml
    newscoop_example_plugin:
        resource: "@NewscoopExamplePluginBundle/Controller/"
        type:     annotation
        prefix:   /


With that configuration in routing.yml Newscoop will be informed about all plugin controllers.

#### Work with views (templates)
As you can see we can return smarty view from controler (as response): 

.. code-block:: php

    return $this->render('NewscoopExamplePluginBundle:Default:index.html.smarty');


of course we can pass data from controller to our view:

.. code-block:: php

    return $this->render('NewscoopExamplePluginBundle:Default:index.html.smarty', array(
        'variable' => 'super extra variable'
    ));


lets go to template:

.. code-block:: smarty

    // ExamplePluginBundle/Resources/views/Default/index.html.smarty
     <h1>this is my variable {{ $variable }} !</h1>


This is very simple template, but we can change this - lets extend our Newscoop default publication theme layout:

This is our default theme page.tpl file 

.. code-block:: smarty

    // ex. newscoop/themes/publication_1/theme_1/page.tpl
    {{ include file="_tpl/_html-head.tpl" }}
    <div id="wrapper">
        {{ include file="_tpl/header.tpl" }}
        <div id="content" class="clearfix">
            <section class="main entry page">
                {{ block content }}{{ /block }}
            </section>
            ...
        </div>
    </div>


And in our plugin template we can do something like this:

.. code-block:: smarty

    {{extends file="page.tpl"}}
    {{block content}}
        <h1>this is my variable {{ $variable }} !</h1>
    {{/block}}
