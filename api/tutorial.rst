Tutorial
==============================

.. |url| replace:: `http://newscoop.aes.sourcefabric.net`
.. |app| replace:: `http://www.example.org`
.. |token| replace:: `2M4OTgxMTM2YWJiMzZmZWNkYTJkZDZlZmY2ZTBiNmUyOTMyZWNlMzNjNDM3NjMzMmU3MWI2OGI4MGM0ODhjNg`


This tutorial shows you how to authenticate a client web application with Newscoop, and how to extract a list of articles that share the same topic. It assumes you have a working Newscoop installation at |url|, your web application is at |app| and you have a valid Newcoop user account.

Pre Authentication Setup
-------------------------

.. So, prerequisites, realistically people are not going to be Newscoop admins, so they will just ask for client id and secret. What is the secret for?

.. But they still need user credentials

Before authenticating your client application, add it to the "Client List" in the `Newscoop Admin interface <http://newscoop.aes.sourcefabric.net/admin/configure-api>`_ . You'll need to provide

* A name for your client application.
* The Newcoop Publication you are accessing.
* The URI the client is redirected to after authentication, which will parse the authentication token from the URL string. 

After adding the client application, make a note of the `Client id` displayed in the table, you'll need it to authenticate.

Authentication
---------------------

To authenticate your client application:

1.  Make a GET request to `/oauth/v2/auth` using the following parameters:

        `client_id`
                YOUR ID, for example `9_1irxa0qcy3ms48c8c8wsgcgsc04k0s0w0g0sg4cco4kocoowoo`

        `redirect_uri`
                YOUR URI, for example |app|. This must match the URI you added in the Newscoop Admin Interface above.  Remember to encode the URI. 

        `response_type`
                `token`

        A full request looks like this::

                http://newscoop.aes.sourcefabric.net/oauth/v2/auth
                ?client_id=9_1irxa0qcy3ms48c8c8wsgcgsc04k0s0w0g0sg4cco4kocoowoo
                &redirect_uri=http%3A%2F%2Fpeter.sourcefabric.net%2F
                &response_type=token

2. Log in to the Newscoop window you are redirected to, and click the `Allow Access` button to authenticate your client application for one hour. You are redirected to the URI you specified in the request, with the following extra parameters:

        `access_token`
                The authentication token to use in your client application requests. 

                For example: `N2M4OTgxMTM2YWJiMzZmZWNkYTJkZDZlZmY2ZTBiNmUyOTMyZWNlMzNjNDM3NjMzMmU3MWI2OGI4MGM0ODhjNg`.

        `expires_in`
                Number of seconds before the authentication token expires. After that you need to request a new one, or refresh the old one. Example `2592000`.

        `token_type`
                Token type according to the `OAUTH 2.0 Authorization Framework <http://tools.ietf.org/html/rfc6749#section-7.1>`_. `bearer`

        `refresh_token`
                Extend the validity of your authentication token for another hour `by refreshing <http://tools.ietf.org/html/rfc6749#page-47>`_. 
               
                Example `NzRlY2E2ODY4MTNhNWVhOTdkMjU2NzgxMWQxOGQ0NzIyYzZmMDYxZGFhYTEwNTkyNWJlNTlmNzg3ZGY4MzAzNA`.

        A full response looks like this::

                http://www.example.org/
                #access_token=N2M4OTgxMTM2YWJiMzZmZWNkYTJkZDZlZmY2ZTBiNmUyOTMyZWNlMzNjNDM3NjMzMmU3MWI2OGI4MGM0ODhjNg
                &expires_in=2592000
                &token_type=bearer
                &refresh_token=NzRlY2E2ODY4MTNhNWVhOTdkMjU2NzgxMWQxOGQ0NzIyYzZmMDYxZGFhYTEwNTkyNWJlNTlmNzg3ZGY4MzAzNA

        If there is an error in authentication, or the URI does not match the URI configured in the Newscoop Admin Interface, the response includes an error code instead::

                http://www.example.org/cb
                #error=access_denied

3. Make sure any further requests, including the ones in the :ref:`getting_a_list` section, include the `access_token` parameter returned in the previous step.

.. _getting_a_list:

Getting a list of Articles
------------------------------

WIP

