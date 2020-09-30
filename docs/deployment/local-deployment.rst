.. _local-deployment:

****************
Local Deployment
****************

.. IMPORTANT::

   This deployment is intended only for testing and demonstration purposes and should not serve for real production use. If you want to provide *FIP Wizard* as a service, visit :ref:`production-deployment`.

Requirements
============

- `Docker Engine <https://docs.docker.com/get-docker/>`_ version 19.03 (or higher)
- `Docker Compose <https://docs.docker.com/compose/install/>`_ version 1.25 (or higher)

Setup
=====

1. `Download <https://github.com/fip-wizard/fip-deployment-basic/archive/master.zip>`_ or ``git clone`` repository https://github.com/fip-wizard/fip-deployment-basic locally
2. Change working directory to the root folder ``fip-deployment-basic``
3. Use `docker-compose <https://docs.docker.com/compose/>`_ to start *FIP Wizard*


.. code-block:: shell

   git clone git@github.com:fip-wizard/fip-deployment-basic.git
   cd fip-deployment-basic
   docker-compose up -d

For additional configuration options, see :ref:`configuration`.

Usage
=====

When *FIP Wizard* is running, you can access the following services:

- http://localhost:8080 - FIP Wizard (DSW)
- http://localhost:10035 - `AllegroGraph <https://blazegraph.com>`_
- http://localhost:27017 - MongoDB (for MongoDB clients)
- http://localhost:3000 - FIP Wizard API

Before you can use the FIP Wizard, you need to create a triple store. Open AllegroGraph in the browser and create ``fip`` triple store there.

For both FIP Wizard, you can use default admin account ``albert.einstein@example.com`` with password ``password``. AllegroGraph and MongoDB are without any authentication.

- To start *FIP Wizard*, use ``docker-compose up -d`` in the root directory.
- To stop *FIP Wizard*, use ``docker-compose down`` in the root directory.
- To restart *FIP Wizard*, use first ``docker-compose down`` and then ``docker-compose up -d`` again.
- To see running services of *FIP Wizard* and their status, use ``docker-compose ps``.
- For debugging and investigating logs, use ``docker-compose logs`` (or ``docker-compose logs -f``).


Update
======

1. Stop *FIP Wizard*
2. Overwrite configurations and ``docker-compose.yml`` or simply ``git pull``
3. Start *FIP Wizard* again


From root directory of ``fip-deployment-basic``:

.. code-block:: shell

   docker-compose down
   git pull
   docker-compose up -d


Notes
=====

For more information about docker-compose and its options, visit `Docker documentation <https://docs.docker.com/compose/>`_.

The main difference with respect to the :ref:`production-deployment` is the absence of proxy and certificates, with opened ports directly instead.
