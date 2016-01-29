Cache your data with Newscoop Cache Service
-------------------------------------------

In Newscoop we have two types of built in caching:

* Templates cache - based on smarty templates cache for static results
* Data cache - caching of mixed values with key on selected cache driver (Array, APC, Memcache, Memcached, Redis)

In this Cookbook chapter we will learn how to use data caching.

First we need to get a cache service instance, which is easy because it's already a service (under "newscoop.cache" name) in our container. 

.. code-block:: php

    // from controller
    $cacheService = $this->container->get('newscoop.cache');

    // from legacy file
    $cacheService = \Zend_Registry::get('container')->get('newscoop.cache');

.. code-block:: yml

    # pass it to service
    ...
    arguments: ["newscoop.cache"]

So we have $cacheService, let's cache some data results:

.. code-block:: php

    // We need to have unique cache key for requested data set
    $cacheId = 'our_unique_cache_id_for_this_data_set';

    // First check if key is already stored in cache
    if ($cacheService->contains($cacheId)) {
        // if it's in cache then use it without making query
        $someResults =  $cacheService->fetch($cacheId);
    } else {
        // if isn't in cache - make query
        $someResults = $em->getRepository('\Newscoop\ExamplePluginBundle\Entity\Example')->findAll();
        // and save results to cache
        $cacheService->save($cacheId, $someResults);
    }

Sometimes you change data set (you add or remove a new example) - then to get fresh data you need to remove the existing cache. Let's see how to do this:

.. code-block:: php

    // We need to know our unique cache key for requested data set
    $cacheId = 'our_unique_cache_id_for_this_data_set';
    $cacheService->delete($cacheId);

If you want to clear the whole driver cache, then just do:

.. code-block:: php

    $cacheDriver = $cacheService->getCacheDriver();
    $cacheDriver->deleteAll();

That's it! Improve your code with data caching!
