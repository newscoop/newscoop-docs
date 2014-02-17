Managing the Plugin Lifecycle
--------------------------------

The best way for plugin lifecycle management is registering event subscriber. Events lifecycle consists of 3 events:

  - plugin.install_vendor_plugin_name
  - plugin.remove_vendor_plugin_name
  - plugin update_vendor_plugin_name
  
``vendor_plugin_name`` is builded from composer name property (vendor/plugin-name). We replace "/" and "-" to "_".

This is example of simple event subscriber class:

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

Next step is registering this class in Event Dispatcher:

.. code-block:: yaml

    // ExamplePluginBundle/Resources/config/services.yml
    services:
        newscoop_example_plugin.lifecyclesubscriber:
            class: Newscoop\ExamplePluginBundle\EventListener\LifecycleSubscriber
            tags:
                - { name: kernel.event_subscriber}

Subscriber can have access for all registered in container services (``php application/console container:debug``), only thing what you must to do is passing services as argument:

.. code-block:: yaml

    // ExamplePluginBundle/Resources/config/services.yml
    services:
        newscoop_example_plugin.lifecyclesubscriber:
            class: Newscoop\ExamplePluginBundle\EventListener\LifecycleSubscriber
            arguments:
                - @em
            tags:
                - { name: kernel.event_subscriber}


and using it in your service (subscriber):

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


In subscriber included in this plugin you can find example of database updating (based on doctrine entities and schema tool)
