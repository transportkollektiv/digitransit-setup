:github_url:  https://github.com/transportkollektiv/digitransit-setup/tree/main/src/datasources.rst

Data Sources
============

This is a list of Digitransit data sources, formats and their capabilities.

.. _`datasources/gtfs`:

GTFS
----

tbd

.. _`datasources/gbfs`:

GBFS
----

GBFS is a data format/API for publishing status information of bikesharing systems, but can also used for kinds of sharing vehicles, like E-scooters and cars. It publishes state and geolocation of available vehicles and stations/docks (but not currently rented vehicles).

Most bike- and scootersharing operators are able to provide GBFS, also most whitelabel sharing software is able to provide GBFS. If sharing operators provide `MDS <https://github.com/openmobilityfoundation/mobility-data-specification>`_ data the authorities, a public GBFS-feed is part of the specification.

The specification is available `on Github <https://github.com/NABSA/gbfs>`_

OpenTripPlanner supports multimodal routing of station based sharing systems with GBFS by default. For free floating systems :ref:`Stadtnavi extension <stadtnavi-extensions/free-floating>`_ can be used.


.. _`datasources/openstreetmap`:

OpenStreetMap
-------------

`OpenStreetMap <openstreetmap.org/>`_ is a crowd sourced geo-database and also the biggest source for open map data. Data from OpenStreetMap is used for foot, bike and car routing in :term:`OpenTripPlanner`.

OpenStreetMap data also can be used for search (geocoding) using :term:`pelias` or :term:`photon` with `photon-pelias-adapter <https://github.com/stadtulm/photon-pelias-adapter>`_.

You most likely don't want the data for the whole planet (which would be available on `planet.openstreetmap.org <https://planet.openstreetmap.org/>`_). Therefore extracts of counties and regions exists. `Geofabrik <https://www.geofabrik.de/en/index.html>`_ Provides daily updated extracts for all countries and some smaller regions on their `download servers <http://download.geofabrik.de/>`_.
Samller Extracts of custom regions can be gererated using the `BBBike extract service <https://extract.bbbike.org/>`_.

For the use in OpenTripPlanner you need to download the data in ``.pbf``-Format (Protocolbuffer).

OpenStreetMap can also be used as background map. For that you need a `Tiles Server <https://wiki.openstreetmap.org/wiki/Tile_servers>`_. If you don't want to setup your own Tile-Server, there are many comercial providers for OpenStreetMap tile servers. Keep in mind that, most free tile severs are funded by donations and should not be used for extensive third party projects.


.. _`datasources/parkapi`:

ParkenDD ParkAPI
-----------------

ParkenDD is a OpenSource Project collecting and scarping parking lot availablity data from different cities and provide them in a unified simple JSON format via `ParkAPI <https://github.com/offenesdresden/ParkAPI>`_.
Data from ParkenDD or in ParkAPI format can be included in digitransit using :ref:`stadtnavi extension dynamic parking lots<stadtnavi-extensions/dynamic-parking-lots>`.

ParkAPI format is also descriped in JSON schema https://github.com/offenesdresden/ParkAPI/wiki/city.json

A Map representation of the data can be found `here <https://parkendd.de/en/map.html#Dresden>`_. You can find the integrated cities in the dropdown on the top right or in the `github repo <https://github.com/offenesdresden/ParkAPI/tree/master/park_api/cities>`_. To integrate new cities you can look a the `Sample_City script <https://github.com/offenesdresden/ParkAPI/blob/master/park_api/cities/Sample_City.py>`_ and the `repos readme <https://github.com/offenesdresden/ParkAPI#adding-support-for-a-new-city>`_.

A public API for all integrated cities is available at  ``https://api.parkendd.de/``.