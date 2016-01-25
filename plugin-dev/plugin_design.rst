Plugin Design
====================

How to write your plugin for better integration with Newscoop.

You can also see `Example Plugin <https://github.com/KnpLabs/KnpMenu>`_ code with all features described in this documentation.

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
        arguments:
            - @translator
        tags:
          - { name: kernel.event_listener, event: newscoop_newscoop.menu_configure, method: onMenuConfigure }

and the menu configuration listener to your plugin:

.. code-block:: php

        <?php
        // EventListener/ConfigureMenuListener.php
        namespace Newscoop\ExamplePluginBundle\EventListener;

        use Newscoop\NewscoopBundle\Event\ConfigureMenuEvent;
        use Symfony\Component\Translation\Translator;

        class ConfigureMenuListener
        {
            protected $translator;

            /**
             * @param Translator $translator
             */
            public function __construct(Translator $translator)
            {
                $this->translator = $translator;
            }

            public function onMenuConfigure(ConfigureMenuEvent $event)
            {
                $menu = $event->getMenu();
                $menu[$this->translator->trans('Plugins')]->addChild(
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



User Permissions in Plugins
---------------------------------

A guide to restricting access to resources in your plugin to certain users. 

Add a `PermissionsListener` class where you define plugin permissions:

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

The first parameter of the `registerPermissions()` method is a custom plugin name, the second parameter is an array of permissions where each key is a unique permission name and each value is a translated permission label.

For example, `plugin_classifieds_edit` is the unique permission name, and the translated permission label should be in the following form::

   plugin.plugin_name.permission_name

Where: 

*  `plugin` - plugin namspace, for example `ads`
* `plugin_name` - plugin name, for example `permissions`
* `permission_name` - permission name, add, delete, etc, for example `edit`

Registering the listener in `services.yml`:

.. code-block:: yaml

    #Acme\DemoPluginBundle\Resources\config\services.yml
    services:
        acme_demo_plugin.permissions.listener:
            class: Acme\DemoPluginBundle\EventListener\PermissionsListener
            arguments:
                - @translator
            tags:
              - { name: kernel.event_listener, event: newscoop.plugins.permissions.register, method: registerPermissions }

To check if a user has given permission, call **hasPermission()** method on `User` object:

.. code-block:: php

    $user->hasPermission('plugin_classifieds_edit');
    
Registering Permissions on Plugin Install/update
+++++++++++++++++++++++++++++++++++++++++++++++++++++++

To register permissions during plugin installation or update process, create a method in `LifecycleSubscriber.php`:

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
 
Then during plugin installation, call the `setPermissions()` method that you created:

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

Checking Permissions in Views - Twig Extension
++++++++++++++++++++++++++++++++++++++++++++++++++

Check user permissions in Twig templates:

.. code-block:: twig

    {% if hasPermission('plugin_classifieds_delete') %}
       <!-- user has delete permission, do some stuff here -->
    {% endif %}


Plugin Cron Jobs
---------------------------------

Newscoop 4.3 introduces a new cron job management system, which also affects repetitive tasks in plugins.

Before Newscoop 4.3, to call a function ever few hours you would create a `Console Command` in the `Acme\ExamplePluginBundle\Command` namespace.

In Newscoop 4.3 you now use `TestCronJobCommand`. The following example prints `Test cron job command.`.

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

To run the cron job, register it on plugin install and update.

Registering Cron Jobs on Plugin Install/Update
++++++++++++++++++++++++++++++++++++++++++++++++

To register a cron job during the plugin install/update process, edit the `LifecycleSubscriber.php` class.

Add the `newscoop.scheduler` service to `LifecycleSubscriber` class.

.. code-block:: twig

    services:
        newscoop_example_plugin.lifecyclesubscriber:
            class: Newscoop\ExamplePluginBundle\EventListener\LifecycleSubscriber
            arguments:
                - @em
                - @newscoop.scheduler

Add a new property called `cronjobs` which is an array of our plugin cron jobs.

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

Use any of the following parameters to define cron jobs:

 - string `command` (**required**) The job to run, either a shell command or an anonymous PHP function. In this example it's our `TestCronJobCommand`
 - string `schedule` (**required**) Crontab schedule format (`man -s 5 crontab`)
 - boolean `enabled` Run this job at scheduled times
 - boolean `debug` Send `scheduler` internal messages to 'debug.log'
 - string `dateFormat` Format for dates on scheduler log messages
 - string `output` Redirect `stdout` and `stderr` to this file
 - string `runOnHost` Run jobs only on this hostname
 - string `environment` Development environment for this job
 - string `runAs` Run as this user, if crontab user has `sudo` privileges

Create a method in the same class to add the cron jobs.

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

And add the `addJobs` methor to the install/update event:

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

Now, when you install the plugin, the `Example plugin test cron job` is inserted into the database, and can be managed via `System Preferences -> Background Jobs Settings`. Plugin 8 in the example screenshot:

.. image:: http://oi61.tinypic.com/p91j4.jpg


Removing Registered Cron Jobs on Plugin Remove Event
+++++++++++++++++++++++++++++++++++++++++++++++++++++++

Cron jobs that are installed by a plugin also need to be removed when the plugin is uninstalled.

Add a function to remove the cron job:

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

and call it during plugin `remove` event:

.. code-block:: php

    <?php
    //Acme\ExamplePluginBundle\EventListener\LifecycleSubscriber.php
    public function remove(GenericEvent $event)
    {
        $tool = new \Doctrine\ORM\Tools\SchemaTool($this->em);
        $tool->dropSchema($this->getClasses(), true);
        $this->removeJobs();
    }
