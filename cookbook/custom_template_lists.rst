Custom template list
--------------------

In Newscoop templates you can use simple lists like ``{{ list_articles }}{{ /list_articles }}`` or ``{{ list_users }}{{ /list_users }}`` to display content. Our lists are builded on top of `Smarty3 plugins block functions <http://www.smarty.net/docsv2/en/plugins.block.functions.tpl>`_.

To create nice list we will need 4 files:
 - NewscoopExampleBundle/Resources/smartyPlugins/block.list_example_content.php - smarty plugin block function
 - NewscoopExampleBundle/TemplateList/ExampleContentCriteria.php - criteria file for list
 - NewscoopExampleBundle/TemplateList/ExampleContentList.php - list
 - NewscoopExampleBundle/EventListener/ListObjectsListener.php - listener with list registration


Lets start form ExampleContentCriteria.php file. Criteria class describe your List for templators, it define properties to be used in list constraints, order and other lists parameters.

Lets asume that our 'Example Content' is an object with properties: ``id, name, description, created_at``. So our criteria object should allow us order list by ``id, name, created_at`` and filter by ``id, name, created_at``. This is our criteria:

.. code-block:: php

    <?php
    /**
     * @package Newscoop\ExamplePluginBundle
     * @author Paweł Mikołajczuk <pawel.mikolajczuk@sourcefabric.org>
     * @copyright 2014 Sourcefabric o.p.s.
     * @license http://www.gnu.org/licenses/gpl-3.0.txt
     */

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
        public $created_at;
    }


Next will be ExampleContentList.php List class is responsible for delivering objects from source to template engine. It must be registered in CampContext class (with ListObjectsListener.php). Lets do it.

.. code-block:: php

    <?php
    /**
     * @package Newscoop\ExamplePluginBundle
     * @author Paweł Mikołajczuk <pawel.mikolajczuk@sourcefabric.org>
     * @copyright 2014 Sourcefabric o.p.s.
     * @license http://www.gnu.org/licenses/gpl-3.0.txt
     */

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

Next file on our todo list is ``block.list_example_content.php``. Block provides special function to our template system, bootstrap our list and configure paginator (with page parameter name).

.. code-block:: php

    <?php
    /**
     * @package Newscoop\ExamplePluginBundle
     * @author Paweł Mikołajczuk <pawel.mikolajczuk@sourcefabric.org>
     * @copyright 2014 Sourcefabric o.p.s.
     * @license http://www.gnu.org/licenses/gpl-3.0.txt
     */

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

        if (!isset($content)) { // init
            $start = $context->next_list_start('\Newscoop\ExamplePluginBundle\TemplateList\ExampleContentList');
            // initiate list object, pass new criteria object and paginatorService
            $list = new \Newscoop\ExamplePluginBundle\TemplateList\ExampleContentList(
                new \Newscoop\ExamplePluginBundle\TemplateList\ExampleContentCriteria(),
                $paginatorService
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


We are almost done! Now we need register in Newscoop our list and new object. For this we will use listener class (``ListObjectsListener.php``).

.. code-block:: php

    <?php
    /**
     * @package Newscoop\ExamplePluginBundle
     * @author Paweł Mikołajczuk <pawel.mikolajczuk@sourcefabric.org>
     * @copyright 2014 Sourcefabric o.p.s.
     * @license http://www.gnu.org/licenses/gpl-3.0.txt
     */

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

Now we need register listener in newscoop.

.. code-block:: yaml

    # Resources/config/services.yml
    newscoop_example_plugin.list_objects.listener:
        class: Newscoop\ExamplePluginBundle\EventListener\ListObjectsListener
        tags:
          - { name: kernel.event_listener, event: newscoop.listobjects.register, method: registerObjects }    


How to use it in template:

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