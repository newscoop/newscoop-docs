Creating Custom Lists for Newcoop Templates
--------------------------------------------

In Newscoop templates you can use Smarty3 lists to display Newscoop content:

.. code:: 

        {{ list_articles }}{{ /list_articles }}
        
        {{ list_users }}{{ /list_users }}

To create custom lists with different Newscoop content, look at the following four files:

List content
        `NewscoopExampleBundle/TemplateList/ExampleContentList.php`

List criteria
        `NewscoopExampleBundle/TemplateList/ExampleContentCriteria.php`

Smarty block
        `NewscoopExampleBundle/Resources/smartyPlugins/block.list_example_content.php`
        
Listener
        `NewscoopExampleBundle/EventListener/ListObjectsListener.php` 

List Content
+++++++++++++++

The List Class delivers the content to the template. It must be registered in the `CampContext` class with `ListObjectsListener.php`. 

.. code-block:: php

    <?php

    namespace Newscoop\ExamplePluginBundle\TemplateList;

    use Newscoop\ListResult;
    use Newscoop\TemplateList\PaginatedBaseList;

    /**
     * ExampleContent List
     */
    class ExampleContentList extends PaginatedBaseList // we extend PaginatedBaseList to use build-in support for paginator
    {
        protected function prepareList($criteria, $parameters)
        {
            // get query builder (or Query), use passed $criteria object to build query
            $target = $this->getListByCriteria($criteria);
            // paginate query builder, pagenumber is injected to paginatorService in list block, use max results from criteria.
            // get ListResults with paginated data

            // if you don't have records in database then you can uncomment this code (it will create dummy criteria objects):
            // $target = array();
            // for ($i=0; $i < 20 ; $i++) {
            //     $target[$i] = new \Newscoop\ExamplePluginBundle\Entity\Example();
            //     $target[$i]->setName('Name for '.$i.' example');
            //     $target[$i]->setDescription('Description for '.$i.' example');
            //     $target[$i]->getCreatedAt(new \DateTime());
            // }


            $list = $this->paginateList($target, null, $criteria->maxResults);

            return $list;
        }

        /**
         * Get list for given criteria
         *
         * You can place this method also in Entity Repository.
         *
         * @param Newscoop\ExamplePluginBundle\TemplateList\ExampleContentCriteria $criteria
         *
         * @return Newscoop\ListResult
         */
        private function getListByCriteria(ExampleContentCriteria $criteria)
        {
            $em = \Zend_Registry::get('container')->get('em');
            $qb = $em->getRepository('Newscoop\ExamplePluginBundle\Entity\Example')
                ->createQueryBuilder('e');

            // use processed by list constraints from list block (template)
            foreach ($criteria->perametersOperators as $key => $operator) {
                $qb->andWhere('e.'.$key.' = :'.$key)
                    ->setParameter($key, $criteria->$key);
            }

            // use processed by list order definitions from list block (template)
            $metadata = $em->getClassMetadata('Newscoop\ExamplePluginBundle\Entity\Example');
            foreach ($criteria->orderBy as $key => $order) {
                if (array_key_exists($key, $metadata->columnNames)) {
                    $key = 'e.' . $key;
                }

                $qb->orderBy($key, $order);
            }

            return $qb;
        }
    }


List Criteria
++++++++++++++++++++++++++++

The Criteria class defines the list properties, constraints, sorting order and other parameters. A custom list for example content with an object with an `id`, `name`, `description` and `created_by_date` should allow sorting and filtering by `id`, `name` and `created_by_date`.

.. code-block:: php

    <?php

    namespace Newscoop\ExamplePluginBundle\TemplateList;

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
        public $created_by_date;
    }

Smarty Block
+++++++++++++++++++

The smarty block is the implementation of the list, template tags and paginator. 

.. code-block:: php

    <?php
    /**
     * list_example_content block plugin
     *
     * Type:     block
     * Name:     list_example_content
     *
     * @param array $params
     * @param mixed $content
     * @param object $smarty
     * @param bool $repeat
     * @return string
     */
    function smarty_block_list_example_content($params, $content, &$smarty, &$repeat)
    {
        $context = $smarty->getTemplateVars('gimme');
        // get paginator service
        $paginatorService = \Zend_Registry::get('container')->get('newscoop.listpaginator.service');
        $cacheService = \Zend_Registry::get('container')->get('newscoop.cache');

        if (!isset($content)) { // init
            $start = $context->next_list_start('\Newscoop\ExamplePluginBundle\TemplateList\ExampleContentList');
            // initiate list object, pass new criteria object and paginatorService
            $list = new \Newscoop\ExamplePluginBundle\TemplateList\ExampleContentList(
                new \Newscoop\ExamplePluginBundle\TemplateList\ExampleContentCriteria(),
                $paginatorService,
                $cacheService
            );

            // inject page parameter name to paginatorService, every list have own name used for pagination
            $list->setPageParameterName($context->next_list_id($context->getListName($list)));
            // inject requested page number (get from request value of list page parameter name)
            $list->setPageNumber(\Zend_Registry::get('container')->get('request')->get($list->getPageParameterName(), 1));

            // get list
            $list->getList($start, $params);
            if ($list->isEmpty()) {
                $context->setCurrentList($list, array());
                $context->resetCurrentList();
                $repeat = false;

                return null;
            }

            // set current list and connect used in list properties
            $context->setCurrentList($list, array('content', 'pagination'));
            // assign current list element to context
            // how we get current_example_content_list name? Our list class have name "ExampleContentList"
            // so we add "current_" and replace all big letters to "_"
            $context->content = $context->current_example_content_list->current;
            $repeat = true;
        } else { // next
            $context->current_example_content_list->defaultIterator()->next();
            if (!is_null($context->current_example_content_list->current)) {
                // assign current list element to context
                $context->content = $context->current_example_content_list->current;
                $repeat = true;
            } else {
                $context->resetCurrentList();
                $repeat = false;
            }
        }

        return $content;
    }

Listener
+++++++++++++++++++++

Register the `List object` in the Newscoop listener class.

.. code-block:: php

    <?php

    namespace Newscoop\ExamplePluginBundle\EventListener;

    use Newscoop\EventDispatcher\Events\CollectObjectsDataEvent;

    class ListObjectsListener
    {
        /**
         * Register plugin list objects in Newscoop
         *
         * @param  CollectObjectsDataEvent $event
         */
        public function registerObjects(CollectObjectsDataEvent $event)
        {
            $event->registerListObject('newscoop\examplepluginbundle\templatelist\examplecontent', array(
                // for newscoop convention we need remove "List" from "ExampleContentList" class name.
                'class' => 'Newscoop\ExamplePluginBundle\TemplateList\ExampleContent',
                // list name without "list_" - another Newscoop convention
                'list' => 'example_content',
                'url_id' => 'cnt',
            ));

            $event->registerObjectTypes('content', array(
                'class' => '\Newscoop\ExamplePluginBundle\Entity\Example'
            ));
        }
    }

And register the listener in the Newscoop configuration.

.. code-block:: yaml

    # Resources/config/services.yml
    newscoop_example_plugin.list_objects.listener:
        class: Newscoop\ExamplePluginBundle\EventListener\ListObjectsListener
        tags:
          - { name: kernel.event_listener, event: newscoop.listobjects.register, method: registerObjects }    


Using the Custom Block in a Template
+++++++++++++++++++++++++++++++++++++++

.. code-block:: smarty

    <ul>
    {{ list_example_content length="2" }}
        <li>
        {{ $gimme->content->getName() }}
        </li>

    {{if $gimme->current_list->at_end}}
    </ul>
    {{ /if }}

        {{ listpagination }}
    {{ /list_example_content }}
