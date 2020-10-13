***************
Usage Scenarios
***************

The **FIP Wizard** can be used in the following scenarios:

FAIR Implementation Profile
---------------------------

In this scenario, we will use FIP Wizard to create and update FAIR Implementation Profile of a community and submit it to the triple store as a nanopublication.

These use cases are require user to be logged in in FIP Wizard:

Create FIP
~~~~~~~~~~

1. Select :guilabel:`Create a FIP` from the left menu
2. Fill-in the name and press :guilabel:`Save` button
3. Fill FIP Questionnaire with your community data, the changes will be saved automatically

Update FIP
~~~~~~~~~~

1. Select :guilabel:`Projects` from the left menu
2. Find by name the FIP you want to edit
3. Fill FIP with new data you have, the changes will be saved automatically

Submit FIP
~~~~~~~~~~

1. Open FIP you want to submit
2. Go to :guilabel:`Documents`
3. Press :guilabel:`New document`
4. Press :guilabel:`Create` (optionally, you can change the document name, e.g. "My community - v0.1")
5. Press three dots on the right for the new document and press :guilabel:`Submit`
6. Select the triple store you want to use and press :guilabel:`Submit`

FAIR Matrix SPARQL Queries
--------------------------

Once you have submitted FIPs in the triple store, you can use various SPARQL queries to explore its contents based on your specific needs. We recommend the `Wikidata's SPARQL tutorial <https://www.wikidata.org/wiki/Wikidata:SPARQL_tutorial>`_.

List declarations for Community
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For a specific Community, e.g. `ENVRI <http://purl.org/np/RAbJisTAUu82wSY_FQ4CrFyA_kPd_0Jvyu2JrNZmO1jPo#ENVRI>`_, you can list all the declarations about current use of a Resource with respect to a FIP Question.

.. code-block:: sparql

   PREFIX fip: <https://w3id.org/fair/fip/terms/>

   SELECT ?decl ?question ?resource
   WHERE {
      ?decl a fip:FIP-Declaration ;
         fip:declared-by <http://purl.org/np/RAbJisTAUu82wSY_FQ4CrFyA_kPd_0Jvyu2JrNZmO1jPo#ENVRI> ;
         fip:refers-to-question ?question ;
         fip:declares-current-use-of ?resource .
   }


List usages of Resource
~~~~~~~~~~~~~~~~~~~~~~~

For a specific Resource, e.g. `Digital Object Identifier <http://www.wikidata.org/entity/Q25670>`_, you can list which communities use (or plan to use) it. You can easily filter our Resources for a specific Community.

.. code-block:: sparql

   PREFIX fip: <https://w3id.org/fair/fip/terms/>

   SELECT DISTINCT ?community
   WHERE {
      {
         ?decl a fip:FIP-Declaration ;
            fip:declared-by ?community ;
            fip:declares-current-use-of <http://www.wikidata.org/entity/Q25670> .
      }
      UNION
      {
         ?decl a fip:FIP-Declaration ;
            fip:declared-by ?community ;
            fip:declares-planned-use-of <http://www.wikidata.org/entity/Q25670> .
      }
   }


Count usages of Resource
~~~~~~~~~~~~~~~~~~~~~~~~

You can also count, for example, how many communities use (currently) a specific Resource.

.. code-block:: sparql

   PREFIX fip: <https://w3id.org/fair/fip/terms/>

   SELECT (COUNT(DISTINCT ?community) as ?count)
   WHERE {
      ?decl a fip:FIP-Declaration ;
         fip:declared-by ?community ;
         fip:declares-current-use-of <http://purl.org/np/RAiyPQd01Y1u-qo3HG3PDVgpHiIuNO9YngYlju1WTyzRI#DOI> .
   }


FAIR Matrix query
~~~~~~~~~~~~~~~~~

This query prepares a table for building FAIR Matrix. You can further limit it by including Community, Question, type of relation (use or planned), or Resource directly in the query.

.. code-block:: text

   PREFIX fip: <https://w3id.org/fair/fip/terms/>

   SELECT ?community ?question ?rel ?resource ?resource_label ?resource_type
   WHERE {
      ?decl a fip:FIP-Declaration ;
         fip:refers-to-question ?question ;
         fip:declared-by ?community ;
         ?rel ?resource .

      VALUES ?rel {
         fip:declares-current-use-of
         fip:declares-planned-use-of
      }
      OPTIONAL { 
         ?resource rdfs:label ?resource_label
      }
      OPTIONAL {
         VALUES ?resource_type {
            fip:Available-FAIR-Enabling-Resource
            fip:FAIR-Enabling-Resource-to-be-Developed
         }
         ?resource a ?resource_type
      }
   }

In FAIR Matrix (or FIP Fingerprint), use of a Resource by a Community can be:

- ``0`` = Resource is not used by Community (cannot be queried, need to compare the list of all possible resources with used resources)
- ``1`` = Resource is currently used by Community (limit only to ``fip:declares-current-use-of``)
- ``2`` = Resource is planned to be used by Community (limit only to ``fip:declares-planned-use-of``)

This query uses `SPARQL 1.1 <https://www.w3.org/TR/sparql11-query/>`_ with keywords ``VALUES`` and ``OPTIONAL``. You need to pre-fill your triple store with the Resources (with type and label at minimum).


