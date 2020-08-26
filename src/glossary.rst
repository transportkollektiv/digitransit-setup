Glossary
========

.. glossary::

  digitransit-ui
    This is the user-facing part of digitransit that provides the user experience. You will probably want to build your localized version yourself. The official version is maintained by HSL and lives at `HSLdevcom/digitransit-ui <https://github.com/HSLdevcom/digitransit-ui>`__

  GTFS
    General Transport Feed Specification, a common exchange format for public (static) transit schedules and related information such as stop locations etcetera. `Wikipedia (en) <https://en.wikipedia.org/wiki/General_Transit_Feed_Specification>`__; `Specification <https://developers.google.com/transit/gtfs/>`__

  GBFS
    General Bikeshare Feed Specification, a common exchange format for bikesharing (or vehicle sharing in general), including the location of sharing docks/hubs and individual free floating vehicles. Conceptually very different from GTFS. `Github specification repo <https://github.com/NABSA/gbfs>`__

  hsl-map-server
    An additional map tile server (`original repository here <https://github.com/HSLdevcom/hsl-map-server>`__) that ingests features such as stops, bikesharing stations etc. and displays them as items on an otherwise transparent map layer, above the basemap. This allows for clickable stops etc.

  OpenStreetMap
    A crowd sourced geo-database. Data from OpenStreetMap is used for foot, bike and car routing in :term:`OpenTripPlanner`. Country and regional data extracts are provided by the `Geofabrik download servers <http://download.geofabrik.de/>`_. For custom regions or smaller extracts the `BBBike extract service <https://extract.bbbike.org/>`_ can be used. (use PBF as extract format.)
    OpenStreetMap data also can be used for search. (:term:`pelias`, :term:`photon`)
    `Website <openstreetmap.org/>`_ 

  OpenTripPlanner
    The Free-/Open-Source multimodal trip planner used for digitransit. `Website <http://www.opentripplanner.org/>`__

  OTP data container
    The Docker container in which the :term:`OpenTripPlanner` routing graph lives. This container can be swapped out for an updated version whenever the need arises.

  ParkenDD / ParkAPI
    OpenSource project to collect and scape parking lot occupancy data and bring them into an `unified json format <https://github.com/offenesdresden/ParkAPI/>`_. Data can be integrated using `dynamic parking lots <stadtnavi-extensions.html#dynamic-parking-lots-routing-and-map-layer>`_.
    `Website <https://parkendd.de>`_, `Github <https://github.com/offenesdresden/ParkAPI/>`_, `Map <https://parkendd.de/map.html>`_

  pelias
    King of Iolcus in Greek mythology. Also, a `geocoder <https://en.wikipedia.org/wiki/Geocoding>`_: 
    insert address, get geocoordinates as a result. Or vice versa. Supports suggest as you type.
    Not maintained anymore, therefore replaced with `photon <https://github.com/komoot/photon>`_ using `photon-pelias-adapter <https://github.com/stadtulm/photon-pelias-adapter>`_ in
    our deployments. `Website <https://www.pelias.io/>`_

  photon
    OpenStreetMap data based `geocoder <https://en.wikipedia.org/wiki/Geocoding>`_. Supports suggest as you type. Based on the `Nominatim Geocoder <https://wiki.openstreetmap.org/wiki/Nominatim>`_.
    We use photon in digitransit with `photon-pelias-adapter <https://github.com/stadtulm/photon-pelias-adapter>`_.
    `Website with Demo <http://photon.komoot.de/>`_, `Github <https://github.com/komoot/photon>`_
