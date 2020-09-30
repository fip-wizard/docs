**********
Components
**********

FIP Wizard consists of the following services:

- `Data Stewardship Wizard (DSW) <https://ds-wizard.org>`_ adjusted to serve as FIP Wizard for creating FAIR Implementation Profiles for the communities.
- AllegroGraph triple store for storing the FIP nanopublications,
- MongoDB used by DSW to store data,
- Submission Service that handles storing FIPs in triple store,
- RabbitMQ for queueing generation of a FIP to a RDF document using DSW document worker,
- (optionally) Nginx proxy for :ref:`production-deployment`.
