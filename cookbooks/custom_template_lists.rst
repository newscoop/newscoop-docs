Introduce custom template list with your plugin
-----------------------------------------------

In Newscoop templates you can use simple lists like ``{{ list_articles }}{{ /list_articles }}`` or ``{{ list_users }}{{ /list_users }}`` to display content. Our lists are builded on top of `Smarty3 plugins block functions <http://www.smarty.net/docsv2/en/plugins.block.functions.tpl>`_.

To create nice list we will need 4 files:
 - NewscoopExampleBundle/Resources/smartyPlugins/block.list_example_content.php - smarty plugin block function
 - NewscoopExampleBundle/TemplateList/ExampleContentCriteria.php - criteria file for list
 - NewscoopExampleBundle/TemplateList/ExampleContentList.php - list
 - NewscoopExampleBundle/EventListener/ListObjectsListener.php - listener with list registration


Lets start form ExampleContentCriteria.php file. Criteria class describe your List for templators, it define properties to be used in list constraints, order and other lists parameters.

Lets asume that our 'Example Content' is an object with properties: ``id, name, message, created_at, is_active``. So our criteria object should allow us order list by ``id, created_at, name`` and filter by `` id, name, created_at, is_active``. This is Our criteria:

.. code-block:: php

    <?php
    namespace Newscoop\ExampleBundle\TemplateList;

    use Newscoop\Criteria;

    class ExampleContentCriteria extends Criteria
    {
        /**
         * @var int
         */
        public $id;

        /**
         * @var string
         */
        public $name;

        /**
         * @var \DateTime
         */
        public $created_at;

        /**
         * @var boolean
         */
        public $is_active;
    }


