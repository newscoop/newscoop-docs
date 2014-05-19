Tutorial
==============================

.. These do not work in code blocks, and so are fairly pointless.

.. |url| replace:: `http://newscoop.aes.sourcefabric.net`
.. |app| replace:: `http%3A%2F%2Fpeter.sourcefabric.net%2F`
.. |token| replace:: `2M4OTgxMTM2YWJiMzZmZWNkYTJkZDZlZmY2ZTBiNmUyOTMyZWNlMzNjNDM3NjMzMmU3MWI2OGI4MGM0ODhjNg`


This tutorial shows you how to authenticate a client web application with Newscoop, and how to extract a list of articles for a particular topic. It assumes you have a working Newscoop installation, you know the URI of your client web application and you have a valid Newcoop user account.

.. What permissions does the Newscoop User need?

Pre Authentication Setup
-------------------------

.. So, prerequisites, realistically people are not going to be Newscoop admins, so they will just ask for client id and secret. What is the secret for?

.. But they still need user credentials

Before your client web application can use the Newscoop RESTful API, you need to add it to the "Client List" in the `Newscoop Admin interface <http://newscoop.aes.sourcefabric.net/admin/configure-api>`_ . You need to provide

* A name for your client web application.
* The Newcoop Publication you are accessing.
* The URI of your client web application which parses the authentication token sent by Newscoop.

After adding the client application, make a note of the `Client id` displayed in the table, you need it to authenticate.

.. _authentication:

Authentication for web applications
-------------------------------------

To authenticate your client web application:

1. Make a GET request to `/oauth/v2/auth` using the following parameters:

        `client_id`
                YOUR ID, for example `9_1irxa0qcy3ms48c8c8wsgcgsc04k0s0w0g0sg4cco4kocoowoo`

        `redirect_uri`
                YOUR URI, for example `http://myapp.example.com/`. This must match the URI you added in the Newscoop Admin Interface above. Remember to encode the URI. 

        `response_type`
                `token`

   A full request looks like this::

       http://newscoop.aes.sourcefabric.net/oauth/v2/auth
       ?client_id=9_1irxa0qcy3ms48c8c8wsgcgsc04k0s0w0g0sg4cco4kocoowoo
       &redirect_uri=http%3A%2F%2Fmyapp.example.com%2F
       &response_type=token

2. Log in to the Newscoop window you are redirected to, and click the `Allow Access` button to authenticate your client web application for one hour. You are redirected to the URI you specified in the request, with the following extra parameters:

        `access_token`
                The authentication token to use in your client application requests. 

                For example: `N2M4OTgxMTM2YWJiMzZmZWNkYTJkZDZlZmY2ZTBiNmUyOTMyZWNlMzNjNDM3NjMzMmU3MWI2OGI4MGM0ODhjNg`.

        `expires_in`
                Number of seconds before the authentication token expires. After that you need to request a new one, or refresh the old one. Example `2592000`.

        `token_type`
                Token type according to the `OAUTH 2.0 Authorization Framework <http://tools.ietf.org/html/rfc6749#section-7.1>`_. `bearer`

        `refresh_token`
                Extend the validity of your authentication token for another hour `by refreshing <http://tools.ietf.org/html/rfc6749#page-47>`_. 
               
                For example: `NzRlY2E2ODY4MTNhNWVhOTdkMjU2NzgxMWQxOGQ0NzIyYzZmMDYxZGFhYTEwNTkyNWJlNTlmNzg3ZGY4MzAzNA`.

   A full response looks like this::

       http://myapp.example.com/
       #access_token=N2M4OTgxMTM2YWJiMzZmZWNkYTJkZDZlZmY2ZTBiNmUyOTMyZWNlMzNjNDM3NjMzMmU3MWI2OGI4MGM0ODhjNg
       &expires_in=2592000
       &token_type=bearer
       &refresh_token=NzRlY2E2ODY4MTNhNWVhOTdkMjU2NzgxMWQxOGQ0NzIyYzZmMDYxZGFhYTEwNTkyNWJlNTlmNzg3ZGY4MzAzNA


   If there is an error in authentication, or the URI does not match the URI configured in the Newscoop Admin Interface, the response includes an error code instead::

       http://myapp.example.com/
       #error=access_denied

3. Make sure any further requests, including the ones in the :ref:`getting_a_list` section, include the `access_token` parameter returned in the previous step.

.. _getting_a_list:

Getting a list of Articles
------------------------------

To get a list of Articles for a particular topic you need to make two requests, one to list the topics, and another to get the articles for a particular topic:

1. Make a GET request to `/rest-api/topics` using the authentication token from :ref:`authentication`.

   The request looks like this::
       
       http://newscoop.aes.sourcefabric.net/content-api/topics
       ?access_token=N2M4OTgxMTM2YWJiMzZmZWNkYTJkZDZlZmY2ZTBiNmUyOTMyZWNlMzNjNDM3NjMzMmU3MWI2OGI4MGM0ODhjNg

   The response looks like this::

        { 
          "items" :  [ 
            { 
              "id" :  395 , 
              "title" :  "Caspar Baader" 
            }, 
            { 
              "id" :  394 , 
              "title" :  "Thomas de Courten" 
            }, 
            { 
              "id" :  268 , 
              "title" :  "Völkermord" 
            } 
          ], 
          "pagination" :  { 
            "itemsPerPage" :  3 , 
            "currentPage" :  1 , 
            "itemsCount" :  771 , 
            "nextPageLink" :  "http://newscoop.aes.sourcefabric.net/content-api/topics?page=2&items_per_page=10" 
          } 
        }

   Note the pagination link at the bottom of the json items array. To look at the second page of results, add your authentication token to that URL and make another GET request.

2. To get a list of topics about Thomas de Courten, for example, make a note of the relevant `id` and make a GET request to `/rest-api/topics/{id}/{language}/articles`, replacing `{id}` with `394` and `{language}` with `de`. 

   .. note:: Currently you need to know the language code of the topic to make the request. You can see what language a topic is written in in the Newscoop Admin Interface.

   ::

    http://newscoop.aes.sourcefabric.net/content-api/topics/394/de/articles
    ?access_token=N2M4OTgxMTM2YWJiMzZmZWNkYTJkZDZlZmY2ZTBiNmUyOTMyZWNlMzNjNDM3NjMzMmU3MWI2OGI4MGM0ODhjNg

   The response contains the topic id and title, and a list of items::

        {
          "id": 3,
          "title": "FC Basel",
          "items": [
            {
              "language": "de",
              "fields": {
                "updated": "",
                "dateline": "Champions League, FC Basel",
                "short_name": "Die kleine Presseschau",
                "seo_title": "So ordnet nationale und internationale Presse den Sieg des FCB ein",

         ...

   A full list of fields in the json response is in the `API reference <http://newscoop.aes.sourcefabric.net/documentation/rest-api/#get--content-api-comments-article-{number}-{language}-{order}-recommended.{_format}>`_.
