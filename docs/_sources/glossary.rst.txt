Glossary
========

.. glossary::

  digitransit-ui
    This is the user-facing part of digitransit that provides the user experience. You will probably want to build your localized version yourself. The official version is maintained by HSL and lives at `HSLdevcom/digitransit-ui <https://github.com/HSLdevcom/digitransit-ui>`__

  GTFS
    General Transport Feed Specification, a common exchange format for public (static) transit schedules and related information such as stop locations etcetera. `Wikipedia (en) <https://en.wikipedia.org/wiki/General_Transit_Feed_Specification>`__; `Specification <https://developers.google.com/transit/gtfs/>`__

  GBFS
    General Bikeshare Feed Specification, a common exchange format for bikesharing (or vehicle sharing in general), including the location of sharing docks/hubs and individual vehicles. Conceptually very different from GTFS. `Github specification repo <https://github.com/NABSA/gbfs>`__

  OpenTripPlanner
    The Free-/Open-Source multimodal trip planner used for digitransit. `Website <http://www.opentripplanner.org/>`__

  OTP Data Container
    The Docker container in which the :term:`OpenTripPlanner` routing graph lives. This container can be swapped out for an updated version whenever the need arises.
