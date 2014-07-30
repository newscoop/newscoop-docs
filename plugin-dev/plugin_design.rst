Plugin Design
====================

How to write your plugin for better integration with Newscoop.

Managing the Plugin Lifecycle
--------------------------------

To manage the plugin from installation to removal, register the following event subscribers: 

  - plugin.install_vendor_plugin_name
  - plugin.remove_vendor_plugin_name
  - plugin update_vendor_plugin_name

``vendor_plugin_name`` is the composer name property (vendor/plugin-name) with slashes `/` and hyphens `-` converted to underscores `_`.

This is an example of an event subscriber class containing placeholder functions for the three events:

.. code-block:: php

    // ExamplePluginBundle/EventListener/LifecycleSubscriber.php
    <?php
    namespace Newscoop\ExamplePluginBundle\EventListener;

    use Symfony\Component\EventDispatcher\EventSubscriberInterface;
    use Newscoop\EventDispatcher\Events\GenericEvent;

    /**
     * Event lifecycle management
     */
    class LifecycleSubscriber implements EventSubscriberInterface
    {
        public function install(GenericEvent $event)
        {
            // do something on install
        }

        public function update(GenericEvent $event)
        {
            // do something on update
        }

        public function remove(GenericEvent $event)
        {
            // do something on remove
        }

        public static function getSubscribedEvents()
        {
            return array(
                'plugin.install.newscoop_example_plugin' => array('install', 1),
                'plugin.update.newscoop_example_plugin' => array('update', 1),
                'plugin.remove.newscoop_example_plugin' => array('remove', 1),
            );
        }
    }

The next step is registering the class in the Event Dispatcher:

.. code-block:: yaml

    // ExamplePluginBundle/Resources/config/services.yml
    services:
        newscoop_example_plugin.lifecyclesubscriber:
            class: Newscoop\ExamplePluginBundle\EventListener\LifecycleSubscriber
            tags:
                - { name: kernel.event_subscriber}

To provide access to all registered container services (``php application/console container:debug``), pass the services as the ``@em`` argument

.. code-block:: yaml

    // ExamplePluginBundle/Resources/config/services.yml
    services:
        newscoop_example_plugin.lifecyclesubscriber:
            class: Newscoop\ExamplePluginBundle\EventListener\LifecycleSubscriber
            arguments:
                - @em
            tags:
                - { name: kernel.event_subscriber}

and use it in your service subscriber:

.. code-block:: php

    // ExamplePluginBundle/EventListener/LifecycleSubscriber.php
    ...
    class LifecycleSubscriber implements EventSubscriberInterface
    {
        private $em;

        public function __construct($em) {
            $this->em = $em;
        }
        ...

..  In subscriber included in this plugin you can find example of database updating (based on doctrine entities and schema tool)


   design/controllers.rst

The Newscoop plugins system is based on the Symfony Bundles system, so almost all Symfony features are available. To create a new controller and route, start by creating the controller class:

.. code-block:: php

        <?php
        // ExamplePluginBundle/Controller/LifecycleSubscriber.php

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
 
Note the annotation for route configuration ``@Route("/testnewscoop")``. Register the controller class in the system:

.. code-block:: yaml

        // ExamplePluginBundle/Resources/config/routing.yml
        newscoop_example_plugin:
            resource: "@NewscoopExamplePluginBundle/Controller/"
            type:     annotation
            prefix:   /

Working with views and templates
+++++++++++++++++++++++++++++++++

The previous Controller example returns a smarty template view:

.. code-block:: php

        return $this->render('NewscoopExamplePluginBundle:Default:index.html.smarty');

You can pass data from the controller to the view:

.. code-block:: php

        return $this->render('NewscoopExamplePluginBundle:Default:index.html.smarty', array(
            'variable' => 'super extra variable'
        ));

The original template is very simple:

.. code-block:: html

        // ExamplePluginBundle/Resources/views/Default/index.html.smarty
        <h1>this is my variable {{ $variable }} !</h1>

For a more complex layout, use the Newscoop default publication theme layout ``page.tpl``:

.. code-block:: html

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

in the plugin template:

.. code-block:: html

        {{extends file="page.tpl"}}
        {{block content}}
            <h1>this is my variable {{ $variable }} !</h1>
        {{/block}}

Creating Database Entities
---------------------------

Newscoop uses `Doctrine2 <http://www.doctrine-project.org/>`_ for database entity management:

* Get the entity manager from the Newscoop container using ``$this->container->get('em');``
* Use the full FQN notation when getting entities: ``$em->getRepository('Newscoop\ExamplePluginBundle\Entity\OurEntity');``


Adding Admin Controllers
---------------------------------

Admin Controllers consist of an action and a route, as in the example in ``Newscoop\ExamplePluginBundle\Controller\DefaultController``. You can use Twig or Smarty as a template engine. There is information on extending the default admin layout, header, menu and footer in ``Resources/views/Default/admin.html.twig``.

Adding a Plugin Menu to the Newscoop Admin Menu
++++++++++++++++++++++++++++++++++++++++++++++++

The Newscoop Admin menu uses the `KNP Menu Library <https://github.com/KnpLabs/KnpMenu>`_ and `KNP MenuBundle <https://github.com/KnpLabs/KnpMenuBundle>`_. To add a Plugin Menu to the Admin Menu, add the service declaration:

.. code-block:: yaml

    newscoop_example_plugin.configure_menu_listener:
        class: Newscoop\ExamplePluginBundle\EventListener\ConfigureMenuListener
        tags:
          - { name: kernel.event_listener, event: newscoop_newscoop.menu_configure, method: onMenuConfigure }

and the menu configuration listener to your plugin:

.. code-block:: php

        <?php
        // EventListener/ConfigureMenuListener.php
        namespace Newscoop\ExamplePluginBundle\EventListener;

        use Newscoop\NewscoopBundle\Event\ConfigureMenuEvent;

        class ConfigureMenuListener
        {
            public function onMenuConfigure(ConfigureMenuEvent $event)
            {
                $menu = $event->getMenu();
                $menu[getGS('Plugins')]->addChild(
                    'Example Plugin', 
                    array('uri' => $event->getRouter()->generate('newscoop_exampleplugin_default_admin'))
                );
            }
        }

Adding Smarty Template Plugins
-------------------------------

The Newscoop template language is Smarty3. Any Smarty3 plugins in 

``<ExamplePluginBundle>/Resources/smartyPlugins``

are automatically loaded and available in your templates.

Adding Dashboard Widgets
-----------------------------

The Newscoop admin panel automatically loads dashboard widgets from:

``<ExamplePluginBundle>/newscoopWidgets``

Plugin Hooks
---------------------

Plugin hooks let you use existing Newscoop functionality in your plugins. Hooks are defined in PHP files in ``<newscoopRoot>/admin-files/``:

* ``issues/edit.php``
* ``sections/edit.php``
* ``articles/edit_html.php``
* ``system_pref/index.php``
* ``system_pref/do_edit.php``
* ``pub/pub_form.php``

Example hook:

.. code-block:: php

        <?php
        //newscoop/admin-files/articles/edit_html.php:

            echo \Zend_Registry::get('container')->getService('newscoop.plugins.service')
                ->renderPluginHooks('newscoop_admin.interface.article.edit.sidebar', null, array(
                    'article' => $articleObj, 
                    'edit_mode' => $f_edit_mode
                ));
        ?>

..
        //newscoop/admin-files/pub/pub_form.php:
        <?php
            echo \Zend_Registry::get('container')->getService('newscoop.plugins.service')
                ->renderPluginHooks('newscoop_admin.interface.publication.edit', null, array(
                    'publication' => $publicationObj
                ));
        ?>


Adding a Plugin Hook to your Plugin
++++++++++++++++++++++++++++++++++++++++++

Define the hook as a service, an addition to the article editing sidebar ``articles/edit_html.php``:

.. code-block:: yaml

        //Resources/config/services.yml
        newscoop_example_plugin.hooks.listener:
                class:     "Newscoop\ExamplePluginBundle\EventListener\HooksListener"
                arguments: ["@service_container"]
                tags:
                  - { name: kernel.event_listener, event: newscoop_admin.interface.article.edit.sidebar, method: sidebar }

In the ``EventListener`` folder of your plugin directory, ``<ExamplePluginBundle>/EventListener`` create ``HooksListener.php`` as specified in ``services.yml`` above:

.. code-block:: php

        <?php

        namespace Newscoop\ExamplePluginBundle\EventListener;

        use Symfony\Component\HttpFoundation\Request;
        use Newscoop\EventDispatcher\Events\PluginHooksEvent;

        class HooksListener
        {
            private $container;

            public function __construct($container)
            {
                $this->container = $container;
            }

            public function sidebar(PluginHooksEvent $event)
            {
                $response = $this->container->get('templating')->renderResponse(
                    'NewscoopExamplePluginBundle:Hooks:sidebar.html.twig',
                    array(
                        'pluginName' => 'ExamplePluginBundle',
                        'info' => 'This is response from plugin hook!'
                    )
                );

                $event->addHookResponse($response);
            }
        }

The ``sidebar()`` method takes a ``PluginHooksEvent`` type as parameter. The `PluginHooksEvent.php <https://github.com/sourcefabric/Newscoop/blob/master/newscoop/library/Newscoop/EventDispatcher/Events/PluginHooksEvent.php>`_ class collects ``Response`` objects from the plugin admin interface hooks.

Next, inside the ``Resources/views`` directory of your plugin create the ``Hooks`` directory we specified in the HooksListener. Then inside the ``Hooks`` directory create the view for the action: ``sidebar.html.twig``.

.. code-block:: html

        <div class="articlebox" title="{{ pluginName }}">
            <p>{{ info }}</p>
        </div>

The plugin response from the hook shows up in the article editing view:

.. image:: http://i41.tinypic.com/16a1j85.png


Plugin permissions
---------------------------------

This guide will help you understand how to set up permissions in your Plugin so you can restrict access for users to some resources. Next, these permissions will be available in Newscoop ACL in Backend.

To register plugin permissions you have to add `PermissionsListener` first, where you will be able to define plugin permissions
e.g.:

.. code-block:: php
        
        <?php
        namespace Acme\DemoPluginBundle\EventListener;
        
        use Newscoop\EventDispatcher\Events\PluginPermissionsEvent;
        use Symfony\Component\Translation\Translator;
        
        class PermissionsListener
        {
            /**
             * Translator
             * @var Translator
             */
             protected $translator;

             public function __construct(Translator $translator)
             {
                 $this->translator = $translator;
             }

             /**
              * Register plugin permissions in Newscoop ACL
              *
              * @param PluginPermissionsEvent $event
              */
             public function registerPermissions(PluginPermissionsEvent $event)
             {
                 $event->registerPermissions($this->translator->trans('ads.menu.name'), array(
                     'plugin_classifieds_edit' => $this->translator->trans('ads.permissions.edit'),
                 ));
             }
         }

First parameter of `registerPermissions()` method is some custom plugin name. Second parameter is array of permissions. Key of this array is unique permission identifier and value is permission translated label.

**Permission unique identifier:**

e.g: `plugin_classifieds_edit` is unique permission name and its structure should be in format as presented below, to register it properly in Newscoop ACL.

.. code-block:: 

* plugin - plugins namspace
* _<plugin_name>_ - plugin name
* <permission_name> - permission name e.g. edit, manage, delete etc.

Next step will be registering our newly created listener in services.yml file:

.. code-block:: yaml

    #Acme\DemoPluginBundle\Resources\config\services.yml
    services:
        acme_demo_plugin.permissions.listener:
            class: Acme\DemoPluginBundle\EventListener\PermissionsListener
            arguments:
                - @translator
            tags:
              - { name: kernel.event_listener, event: newscoop.plugins.permissions.register, method: registerPermissions }

To simply check if user has given permission you have to invoke **hasPermission()** method on User object:

.. code-block:: php

    $user->hasPermission('plugin_classifieds_edit');
    
Register permissions on plugin install/update event
++++++++++++++++++++++++++++++++++++++++++

To register permissions in Newscoop during the plugin install/update process you will need to create inside `LifecycleSubscriber.php` class, method:

.. code-block:: php
    
    <?php
    //Acme\DemoPluginBundle\EventListener\LifecycleSubscriber.php

    /**
     * Collect plugin permissions
     */
    private function setPermissions()
    {
        $this->pluginsService->savePluginPermissions($this->pluginsService->collectPermissions($this->translator->trans('ads.menu.name')));
    }
    
Then on install method you can call method that you created:

.. code-block:: php
  
    <?php
    //Acme\DemoPluginBundle\EventListener\LifecycleSubscriber.php

    public function install(GenericEvent $event)
    {
        $tool = new \Doctrine\ORM\Tools\SchemaTool($this->em);
        $tool->updateSchema($this->getClasses(), true);

        $this->em->getProxyFactory()->generateProxyClasses($this->getClasses(), __DIR__ . '/../../../../library/Proxy');
        $this->setPermissions();
    }

Plugin permissions in views - Twig Extension
++++++++++++++++++++++++++++++++++++++++++

We have also created Twig extensions so you can easly check for user permissions in Twig templates easly.
Example usage:

.. code-block:: twig

    {% if hasPermission('plugin_classifieds_delete') %}
       <!-- user has delete permission, do some stuff here -->
    {% endif %}


Plugin cron jobs
---------------------------------

In Newscoop 4.3 we have introduced a new way to handle cron jobs management which also affect plugins. This guide will help you to register/remove your plugin cron job(s).

Okay, let's start. Imagine that you want to call some operations from within your plugin every few minutes, hours etc. which will for example update some data in database. In this case you would normally create `Console Command` which will be under `Acme\ExamplePluginBundle\Command` namespace.

In this example let's use `TestCronJobCommand` which prints `Test cron job command.` text on screen. (see below).

.. code-block:: php

    <?php

    namespace Acme\ExamplePluginBundle\Command;

    use Symfony\Component\Console;
    use Symfony\Bundle\FrameworkBundle\Command\ContainerAwareCommand;

    /**
     * Test cron job command
     */
    class TestCronJobCommand extends ContainerAwareCommand
    {
        /**
         */
        protected function configure()
        {
            $this->setName('example:test')
                ->setDescription('Example test cron job command');
        }

        /**
         */
        protected function execute(Console\Input\InputInterface $input, Console\Output\OutputInterface $output)
        {
            try {
                $output->writeln('<info>Test cron job command.</info>');
            } catch (\Exception $e) {
                $output->writeln('<error>Error occured: '.$e->getMessage().'</error>');

                return false;
            }
        }
    }

We have our console command class which will be representing our cron job. Next step that we should do is to register new cron jobs (console commands) on plugin install/update event.

Register cron jobs on plugin install/update event
++++++++++++++++++++++++++++++++++++++++++

To register cron job(s) in Newscoop during the plugin install/update process we will need to add some extra properties, methods etc., inside `LifecycleSubscriber.php` class.

Frist of all we have to add `SchedulerService` to `LifecycleSubscriber` constructor and define our cron jobs:

.. code-block:: twig

    services:
        newscoop_example_plugin.lifecyclesubscriber:
            class: Newscoop\ExamplePluginBundle\EventListener\LifecycleSubscriber
            arguments:
                - @em
                - @newscoop.scheduler

As you can see we have added `newscoop.scheduler` service to `LifecycleSubscriber` class. Let's define cron jobs now:

.. code-block:: php

    <?php
    //Acme\ExamplePluginBundle\EventListener\LifecycleSubscriber.php

    protected $scheduler;

    protected $cronjobs;

    public function __construct(EntityManager $em, SchedulerService $scheduler)
    {
        $appDirectory = realpath(__DIR__.'/../../../../application/console');
        $this->em = $em;
        $this->scheduler = $scheduler;
        $this->cronjobs = array(
            "Example plugin test cron job" => array(
                'command' => $appDirectory . ' example:test',
                'schedule' => '* * * * *',
            ),
            /*"Another test cron job" => array(
                'command' => $appDirectory . ' example:anothertest',
                'schedule' => '* * * * *',
            ),*/
        );
    }

As you can see above we have added new property called `cronjobs` which is an array of our plugin cron jobs. `Example plugin test cron job` is custom name of our cron job. Value of given key is array of cron job parameters which you can customize however you like (see the full list of parameters below).

**Full list of parameters:**
 - string `command` The job to run (either a shell command or anonymous PHP function) (required) - in this example it's our `TestCronJobCommand`
 - string `schedule` Crontab schedule format (`man -s 5 crontab`) (required)
 - boolean `enabled` Run this job at scheduled times
 - boolean `debug` Send `scheduler` internal messages to 'debug.log'
 - string `dateFormat` Format for dates on scheduler log messages
 - string `output` Redirect `stdout` and `stderr` to this file
 - string `runOnHost` Run jobs only on this hostname
 - string `environment` Development environment for this job
 - string `runAs` Run as this user, if crontab user has `sudo` privileges

Next, we have to create method in same class to add our cron jobs so they can be executed by given schedule.

.. code-block:: php

    <?php
    //Acme\ExamplePluginBundle\EventListener\LifecycleSubscriber.php

    /**
     * Add plugin cron jobs
     */
    private function addJobs()
    {
        foreach ($this->cronjobs as $jobName => $jobConfig) {
            $this->scheduler->registerJob($jobName, $jobConfig);
        }
    }

Then on install/update method you can add our newly creaded `addJobs` method.

.. code-block:: php

    <?php
    //Acme\DemoPluginBundle\EventListener\LifecycleSubscriber.php

    public function install(GenericEvent $event)
    {
        $tool = new \Doctrine\ORM\Tools\SchemaTool($this->em);
        $tool->updateSchema($this->getClasses(), true);

        $this->em->getProxyFactory()->generateProxyClasses($this->getClasses(), __DIR__ . '/../../../../library/Proxy');
        $this->addJobs();
    }

After plugin install process, `Example plugin test cron job` will be inserted into database and you will be able to manage it via `System Preferences -> Background Jobs Settings`

![System Preferences -> Background Jobs Settings](http://oi62.tinypic.com/123prbs.jpg)

New plugin cron job is visible on 8th position.

Remove registered cron jobs on plugin remove event
++++++++++++++++++++++++++++++++++++++++++

When you will want to uninstall your plugin we should also get rid of cron jobs that are not used anymore and were registered when you installed your plugin. To do so we will have to add extra method to remove these cron jobs.

.. code-block:: php

    <?php
    //Acme\ExamplePluginBundle\EventListener\LifecycleSubscriber.php

    /**
     * Remove plugin cron jobs
     */
    private function removeJobs()
    {
        foreach ($this->cronjobs as $jobName => $jobConfig) {
            $this->scheduler->removeJob($jobName, $jobConfig);
        }
    }

and call it on `remove` event:

.. code-block:: php

    <?php
    //Acme\ExamplePluginBundle\EventListener\LifecycleSubscriber.php
    public function remove(GenericEvent $event)
    {
        $tool = new \Doctrine\ORM\Tools\SchemaTool($this->em);
        $tool->dropSchema($this->getClasses(), true);
        $this->removeJobs();
    }

When we will uninstall plugin, all cron jobs will be automatically removed.