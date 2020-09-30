.. _production-deployment:

*********************
Production Deployment
*********************

.. IMPORTANT::

   This deployment is intended for production use. If you want to just test *FIP Wizard* locally, visit :ref:`local-deployment`.

Requirements
============

- `Docker Engine <https://docs.docker.com/get-docker/>`_ version 19.03 (or higher)
- `Docker Compose <https://docs.docker.com/compose/install/>`_ version 1.25 (or higher)
- Domain and DNS records set for providing *FIP Wizard* (you can use any other subdomains):

  - ``dsw.your-domain.tld`` - for FIP Wizard (DSW)
  - ``api.dsw.your-domain.tld`` - for FIP Wizard API (DSW API)
  - ``sparql.your-domain.tld`` - for Triple Store (Nanopublications)
  
- `certbot <https://certbot.eff.org>`_

Setup
=====

Get FIP Wizard
--------------

`Download <https://github.com/fip-wizard/fip-deployment-production/archive/master.zip>`_ or ``git clone`` repository https://github.com/fip-wizard/fip-deployment-production locally.

The folder ``fip-deployment-production`` we call *FIP Wizard* root directory. It consists all necessary configuration files and ``docker-compose.yml``.

Configure domains and secrets
-----------------------------

There are several things that you need to configure before running *FIP Wizard* for production deployment. In files, look for comments marked with ``(!)``:

1. ``server_name`` and ``ssl_certificate`` values in ``proxy/nginx/agraph.conf`` and ``proxy/nginx/dsw.conf`` with your domain names. Those need to have valid DNS records pointing to that server.
2. ``docker-compose.yml`` -  ``API_URL`` (``dsw_client`` service) to your value for ``api.dsw.your-domain.tld``
3. ``dsw-server/application.yml`` - ``clientUrl`` to your value for  ``dsw.your-domain.tld``, then ``secret``, ``serviceToken``, and ``email`` section according to the comments there
4. ``allegrograph/agraph.cfg`` - set **strong password** and optionally change username using ``SuperUser`` directive, the same credentials must be configured in ``submission-service/config.yml``

Obtain SSL certificates
-----------------------

Before providing *FIP Wizard* you need also to get SSL certificates to be able to use HTTPS. We recommend using Let's Encrypt but you can use any other way and change Nginx proxy configuration accordingly.

1. Comment out ``include`` lines at the end of ``proxy/nginx/nginx.conf``
2. Start the proxy service

.. code-block:: shell

  docker-compose up -d proxy

3. Get certificates for your domains:

.. code-block:: shell

  sudo certbot certonly --webroot -w ./proxy/letsencrypt -d dsw.your-domain.tld

.. code-block:: shell

  sudo certbot certonly --webroot -w ./proxy/letsencrypt -d api.dsw.your-domain.tld

.. code-block:: shell

  sudo certbot certonly --webroot -w ./proxy/letsencrypt -d sparql.your-domain.tld

4. Create certificate file for AllegroGraph (it needs to merge ``cert.pem`` and ``privkey.pem`` obtained by Let's Encrypt into a single file):

.. code-block:: shell

  sudo cat /etc/letsencrypt/live/sparql.your-domain.tld/cert.pem  /etc/letsencrypt/live/sparql.your-domain.tld/privkey.pem > ./allegrograph/cert.pem

5. Stop the proxy service

.. code-block:: shell

  docker-compose down

6. Uncomment lines at the end of ``proxy/nginx/nginx.conf``

If getting certificates fails, it can be caused by incorrectly set DNS records. Optionally, verify if Nginx container is running and view its logs. You should also setup certificates renewal according to `Certbot documentation <https://certbot.eff.org/docs/using.html#renewing-certificates>`_.

First start
-----------

1. Start *FIP Wizard* (and wait a bit until all services start).

.. code-block:: shell

   docker-compose up -d

2. Navigate to ``dsw.your-domain.tld``, login using ``albert.einstein@example.com`` with password ``password`` and change default user accounts with **strong passwords**.
3. In ``sparql.your-domain.tld``, create a repository ``fip`` in catalog ``/`` and create other users with permissions according to your needs (see `AllegroGraph documentation <https://franz.com/agraph/support/documentation/current/managing-users.html#Managing-users-with-AGWebView:-general-comments>`_ for details). For example, create an *anonymous* user with only *read* permissions to catalog */* and repository *fip*.
4. Restart *FIP Wizard* and wait a bit until all services start up (depending on your hardware, less than a minute).

.. code-block:: shell

   docker-compose down
   docker-compose up -d

8. Verify setup by creating FAIR Implementation Profile, saving it, creating a document, and submitting a nanopublication.

🎉 After this, your *FIP Wizard* is ready to be used!

To check if everything is working, you can use ``docker-compose logs`` and ``docker-compose ps`` commands.

⚙️ For additional configuration options, see :ref:`configuration`.

Update
======

1. Stop *FIP Wizard*
2. Overwrite configurations and ``docker-compose.yml`` or simply ``git pull``
3. Check if there are new configuration values to be changed according to your setup (marked with ``(!)`` comments)
4. Start *FIP Wizard* again


From root directory of ``fip-deployment-production``:

.. code-block:: shell

   docker-compose down
   git pull
   docker-compose up -d

This may need you to ``git stash`` your changes and then ``git stash pop`` them (and eventually solve git conflicts).

Notes
=====

For more information about docker-compose and its options, visit `Docker documentation <https://docs.docker.com/compose/>`_.

The main difference with respect to the :ref:`local-deployment` is the adding Nginx proxy, certificates, and other additional security.
