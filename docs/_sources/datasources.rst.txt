:github_url:  https://github.com/transportkollektiv/digitransit-setup/tree/main/src/datasources.rst

Data Sources
============

This is a list of Digitransit data sources, formats and their capabilities.

.. _`datasources/gtfs`:

GTFS
----

`GTFS (General Transport Feed Specification) <https://en.wikipedia.org/wiki/General_Transit_Feed_Specification>`_ is a common exchange format for public (static) transit schedules and related information such as stop locations etcetera. It is the default format OpenTripPlanner is using for public transport routing.

For live updates like vehicle positions, delays and temporary service disruptions `GTFS-RT <https://gtfs.org/reference/realtime/v2/>`_ (GTFS Realtime) can be used as an addition to GTFS. OpenTripPlanner is able to process GTFS-RT, if trip IDs are matching the GTFS-feeds IDs.

If public transport agencies provide their data to other external services like Google Maps or Apple Maps, they are definitely able to provide GTFS-data.

If OTP/digitransit only shows straight lines between trips, the ``shapes.txt`` file is missing in the GTFS. Not all agencies are able to export them or can't export them for copyright reasons. The tool `pfaedle <https://github.com/ad-freiburg/pfaedle>`_ is able to add these shapes to an existing GTFS file. It also looks like `ArcGIS is able <http://www.arcgis.com/home/item.html?id=ce51601e18ae4367814a8a53a1659b33>`_ to generate the ``shapes.txt`` file from GTFS.

.. _`datasources/gbfs`:

GBFS
----

GBFS is a data format/API for publishing status information of bikesharing systems, but can also used for kinds of sharing vehicles, like E-scooters and cars. It publishes state and geolocation of available vehicles and stations/docks (but not currently rented vehicles).

Most bike- and scootersharing operators are able to provide GBFS, also most whitelabel sharing software is able to provide GBFS. If sharing operators provide `MDS <https://github.com/openmobilityfoundation/mobility-data-specification>`_ data the authorities, a public GBFS-feed is part of the specification.

The specification is available `on Github <https://github.com/NABSA/gbfs>`_

OpenTripPlanner supports multimodal routing of station based sharing systems with GBFS by default. For free floating systems :ref:`Stadtnavi extension <stadtnavi-extensions/free-floating>` can be used.


.. _`datasources/openstreetmap`:

OpenStreetMap
-------------

`OpenStreetMap <https://openstreetmap.org/>`_ is a crowd sourced geo-database and also the biggest source for open map data. Data from OpenStreetMap is used for foot, bike and car routing in :term:`OpenTripPlanner`.

OpenStreetMap data also can be used for search (`geocoding <https://en.wikipedia.org/wiki/Geocoding>`_) using :term:`pelias` or :term:`photon` with `photon-pelias-adapter <https://github.com/stadtulm/photon-pelias-adapter>`_.

You most likely don't want the data for the whole planet (which would be available on `planet.openstreetmap.org <https://planet.openstreetmap.org/>`_). Therefore extracts of counties and regions exists. `Geofabrik <https://www.geofabrik.de/en/index.html>`_ provides daily updated extracts for all countries and some smaller regions on their `download servers <http://download.geofabrik.de/>`_.
Samller Extracts of custom regions can be generated using the `BBBike extract service <https://extract.bbbike.org/>`_.

For the use in OpenTripPlanner you need to download the data in ``.pbf``-Format (Protocolbuffer).

OpenStreetMap can also be used as background map. For that you need a `Tile Server <https://wiki.openstreetmap.org/wiki/Tile_servers>`_. If you don't want to setup your own Tile Server, there are many commercial providers for OpenStreetMap tile servers. Keep in mind that, most free tile severs are funded by donations and should not be used for extensive third party projects.


.. _`datasources/parkapi`:

ParkenDD ParkAPI
-----------------

ParkenDD is a OpenSource Project collecting and scarping parking lot availablity data from different cities and provide them in a unified simple JSON format via `ParkAPI <https://github.com/offenesdresden/ParkAPI>`_.
Data from ParkenDD or in ParkAPI format can be included in digitransit using :ref:`stadtnavi extension dynamic parking lots<stadtnavi-extensions/dynamic-parking-lots>`.

ParkAPI format is also descriped in JSON schema https://github.com/offenesdresden/ParkAPI/wiki/city.json

A map representation of the data can be found `here <https://parkendd.de/en/map.html#Dresden>`_. You can find the integrated cities in the dropdown on the top right or in the `github repo <https://github.com/offenesdresden/ParkAPI/tree/master/park_api/cities>`_. To integrate new cities you can look a the `Sample_City script <https://github.com/offenesdresden/ParkAPI/blob/master/park_api/cities/Sample_City.py>`_ and the `repos readme <https://github.com/offenesdresden/ParkAPI#adding-support-for-a-new-city>`_.

A public API for all integrated cities is available at  ``https://api.parkendd.de/``.


GeoJSON
-------

GeoJSON is a popular Geodata format. GeoJSON supports the geometry types Point, LineString and Polygon. in GeoJSON each geometry can have custom properties including complex JSON structures. In digitransit GeoJSON-layers are a easy way to add custom geodata to the map. Examples are charging stations for electic vehicles or ticket zones. GeoJSON layers also support popups with custom content for each geometry.
Custom styling can be added directly into the GeoJSON file.

Be aware, that the complete GeoJSON file will be donwloaded by client. So GeoJSON should be only used for smaller layers and should not exceed a few hundred geometries. Also be aware, that external GeoJSON files needs to be deleiverd with CORS HTTP header. If you are using dynamic generated GeoJSON from external services, you should consider a server side caching proxy for that file. 

To add a GeoJSON Layer to digitransit you only need to add the config to your ``app/configurations/config.XXX.js`` file in digitransit-ui.

A simple example would be

  .. code-block::

    ...,
    geoJson: {
      layers: [
        {
          name: {
            de: 'Ladesäulen',
            en: 'Charging Stations',
          },
          // web address of the data source
          url: 'http://<URL-TO-GEOJSON>/stations.geojson',
        }
      ]
    }


Complete documentation including styling can be found in `docs/GeoJSON <https://github.com/HSLdevcom/digitransit-ui/blob/master/docs/GeoJson.md>`_. For dynamic styling of the GeoJSON, you could manipulate the styling properties based on the data properties with a small external script before delivering the file to digitransit-ui. 

Caution: Styling and popup-content behaves different if GeoJSON contains geometry types other than points. If the file only contians points, property ``address`` will be used for popup content ignoring HTML. If it contains other geometry types then points, popup content from property ``popupContent`` will be rendered as HTML and the geometries are rendered on all zoomlayers.
