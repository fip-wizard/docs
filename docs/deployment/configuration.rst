.. _configuration:

**********************
Advanced Configuration
**********************

To work with *FIP Wizard* you are not required to change anything in the included ``docker-compose.yml`` nor configuration files. For some specific use cases you might want to make some of the following changes.

Persistence
===========

In the basic setup, persistence is assured using mounted folders (bind mounts):

- ``./mongo/data`` - for MongoDB
- ``./allegrograph/data`` - for AllegroGraph triple store (used as nanopublications data storage)

This allows you to easily work with data used by *FIP Wizard*. For example, you can clear those folders (while it is not running) to start over. In some cases you might want to use `Docker volumes <https://docs.docker.com/storage/volumes/>`_ instead. Using Docker volumes is recommended when using Docker for Windows due to common problems related to mounting Windows folders into Linux containers.

.. code-block:: yaml

   # ...

   mongo:
     image: mongo:4.2.3
     restart: always
     ports:
       - 27017:27017
     environment:
       MONGO_INITDB_DATABASE: wizard
     volumes:
       - mongo-data:/data/db  # <- USING DOCKER VOLUME
       - ./mongo/init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js:ro

   # ...

   agraph:
     image: franzinc/agraph:v7.0.1
     restart: always
     ports:
       - 10000-10035:10000-10035
     hostname: agraph
     shm_size: '1gb'
     volumes:
       - agraph-data:/agraph/data    # <- USING DOCKER VOLUME
       - ./allegrograph/agraph.cfg:/agraph/etc/agraph.cfg

   # ...

   volumes:
     mongo-data:
     agraph-data:


To avoid persistence totally (i.e. all data will be lost after ``docker-compose down``). Just comment out or delete lines related to mounting volumes in ``docker-compose.yml```:

.. code-block:: yaml

   # ...

   mongo:
     image: mongo:4.2.3
     restart: always
     ports:
       - 27017:27017
     environment:
       MONGO_INITDB_DATABASE: wizard
     volumes:
       # - mongo-data:/data/db  # <-
       - ./mongo/init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js:ro

   # ...

   agraph:
     image: franzinc/agraph:v7.0.1
     restart: always
     ports:
       - 10000-10035:10000-10035
     hostname: agraph
     shm_size: '1gb'
     volumes:
       # - agraph-data:/agraph/data    # <- 
       - ./allegrograph/agraph.cfg:/agraph/etc/agraph.cfg


.. IMPORTANT::

   Data backups are your responsibility. It is recommended to backup regularly all mounted volumes and store such backups in different site(s).


Changing ports
==============

If you need to change ports because you already use those for other services, you just need to adjust the mappings in ``docker-compose.yml`` file. For example, if you want to access MongoDB on other port than ``27017`` change the mapping ``27017:27017`` to something else, e.g. ``27020:27017``.

.. code-block:: yaml

   # ...

   mongo:
     image: mongo:4.2.3
     restart: always
     ports:
       - 27020:27017
     environment:
       MONGO_INITDB_DATABASE: wizard
     volumes:
       # ...
       - ./mongo/init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js:ro



FIP Wizard emails
=================

There is optional configuration in ``dsw-server/application.yml`` related to email server. You need that to enable:

- User registrations with email-based verification: upon registration a verification email is sent, otherwise administrator have to set new accounts as *Active* manually in users administration.
- Password recovery: when someone forgots password, they can ask for reset link that will be sent to their email address, otherwise it can be again changes only by administrators.

To make those emails working, fill the configuration with your SMTP server and accoung. We recommend using secured emails with SSL/TLS or STARTTLS. For more information, visit `DSW documentation <https://docs.ds-wizard.org/en/latest/admin/configuration.html#mail>`_.

.. NOTE::

   Registrations can be turned off using :guilabel:`Settings` and :guilabel:`Authentication`.
