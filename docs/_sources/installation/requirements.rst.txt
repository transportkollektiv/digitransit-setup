:github_url:  https://github.com/transportkollektiv/digitransit-setup/tree/main/src/installation/requirements.rst

Requirements
============

To build and run your own digitransit instance, you need:

-  One host machine with a-little-bit-more-than-average RAM (i.e. 16+ GB) for
   building the :term:`OTP data container` and :term:`digitransit-ui`
-  One Kubernetes Cluster (can be single-node) with 8+ GB RAM
-  `kubectl <https://kubernetes.io/docs/tasks/tools/install-kubectl/>`_
   installed and configured on your (personal) device for controlling
   said Kubernetes Cluster. (The following tasks _can_ be performed without
   kubectl, but we highly discourage it.)
-  One OpenStreetMap extract for your region (see:
   https://download.geofabrik.de)
-  One (or more) :term:`GTFS` Feeds for public transit routes
-  A GitHub Account (for forking stuff)
-  A Docker Hub Account (for publishing your containers)
-  One Domain and access to DNS for creating a subdomain
-  An externally running `photon <https://github.com/komoot/photon>`_.
   For development and tests, you could try the `public photon instance <https://photon.komoot.de>`_.

optional
~~~~~~~~

-  One (or more) :term:`GBFS` Feeds (for bikesharing and/or carsharing)