:github_url:  https://github.com/transportkollektiv/digitransit-setup/tree/main/src/installation/step-by-step.rst

Step-by-step guide
==================

.. note::
    These seem like a lot of steps. It may be possible to skip some of them or replace
    them with other components (eg. digitransit-proxy) - but for a working instance
    that is easier to maintain, the following steps have proven to be successful. 


0. Preparing your build host
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You need on your build host:

- docker-ce runtime (`installation <https://docs.docker.com/engine/install/ubuntu/>`__, `short installation script <https://get.docker.com>`__)
- git
- nodejs, LTS version (`nodesource repository <https://github.com/nodesource/distributions#deb>`__)
- yarn (`installation <https://yarnpkg.com/getting-started/install>`__)
- build-essential (for the node dependencies)

.. note:: This should be enhanced with copy-pasteable commands

1. Building an OpenTripPlanner Graph
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

At the end of this part, you will end up with a working :term:`OTP data container`. The process works by providing :term:`OpenTripPlanner` with the necessary base data and using it to build the graph that will later be used by OTP to perform routing queries against.

.. important::
    Make sure that the graph is being built with the same version
    of OpenTripPlanner that will later on be used to perform the
    actual queries.

You need: 

- One (or more) :term:`GTFS` Feed with a publicly accessible URL
  (if you only have a :term:`GTFS` zip file, upload it somewhere
  public)
- One (or more) OpenStreetMap extract(s) for your region in .pbf format,
  accessible at a public URL - try https://download.geofabrik.de 

For the configuration, find and memorize a short identifier. You
need this often. In our example, we use ``ulm``.

Method 1: vsh-style modifying of opentripplanner-data-container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. warning:: 
    This method is deprecated as of 2020-06 and is only
    preserved for archival reasons. Please skip ahead to 
    `Method 2: muenster-style custom container`_

.. container:: toggle

    Check out `HSLdevcom/OpenTripPlanner-data-container <https://github.com/HSLdevcom/OpenTripPlanner-data-container>`__

    ``git clone https://github.com/HSLdevcom/OpenTripPlanner-data-container``

    Copy the ``router-waltti`` folder to ``router-ulm`` (replace ``ulm``
    with your :term:`GTFS` identifier). Inside ``router-ulm``, edit
    ``build-config.js``

    ::

        {
          "areaVisibility": true,
          "parentStopLinking": true,
          "osmWayPropertySet": "default",
          "elevationUnitMultiplier": 0.1
        }

    (Remove stuff like ``"fares": "HSL",``, this is not relevant outside of
    finland).

    Also edit ``router-config.js``. The ``routingDefaults`` are mostly okay.
    If you are not satisfied with the routing suggestions, try to modify
    these values. ``updaters`` are for realtime updates to feeds (think:
    GTFS-RT) or :term:`GBFS` (Bikesharing, Carsharing) status updates. If you don't
    have these, simply replace it with ``updaters: []``. Your
    ``router-config.js`` could look like this:

    ::

        {
          "routingDefaults": {
              "walkSpeed": 1.3,
              "transferSlack": 120,
              "maxTransfers": 4,
              "waitReluctance": 0.95,
              "waitAtBeginningFactor": 0.7,
              "walkReluctance": 1.75,
              "stairsReluctance": 1.65,
              "walkBoardCost": 540,
              "itineraryFiltering": 1.0,
              "maxSlope": 0.125
          },
          "updaters": []
        }

    In the main directory, edit the ``config.js`` and add a new
    ``ULM_CONFIG`` like the ``HSL_CONFIG``. Insert your :term:`GTFS` URL. For
    example like this:

    ::

        const ULM_CONFIG = {
          'id': 'ulm',
          'src': [
            src('DING', 'https://www.nvbw.de/fileadmin/nvbw/open-data/Fahrplandaten_mit_Liniennetz/ding.zip', false),
          ],
          'osm': 'ulm',
          // 'dem': 'hsl' // we don't have a Digital Elevation Model
        }

    In the ``setCurrentConfig`` method, you need to add your thusly created
    config to ``ALL_CONFIGS``, like this:

    ::

        const setCurrentConfig = (name) => {
          ALL_CONFIGS = [WALTTI_CONFIG, HSL_CONFIG, FINLAND_CONFIG, ULM_CONFIG].reduce((acc, nxt) => {


    If you use multiple OSM extracts, merge them with tools like `osmium merge <https://gis.stackexchange.com/questions/242704/merging-osm-pbf-files>`__ first.

    Add your OSM extract to the osm config near the end of the file:

    ::

        const osm = [
          { id: 'finland', url: 'https://karttapalvelu.storage.hsldev.com/finland.osm/finland.osm.pbf' },
          { id: 'hsl', url: 'https://karttapalvelu.storage.hsldev.com/hsl.osm/hsl.osm.pbf' },
          { id: 'ulm', url: 'https://download.geofabrik.de/europe/germany/baden-wuerttemberg/tuebingen-regbez-latest.osm.pbf' }
        ]

    Modify ``Dockerfile`` to include your ``router-ulm`` directory: ``ADD router-hsl /opt/otp-data-builder/router-hsl ADD router-waltti /opt/otp-data-builder/router-waltti ADD router-ulm /opt/otp-data-builder/router-ulm``

    Modify ``gulpfile.js`` to include your router configuration in the build
    process. Near the end of the file,
    ``gulp.task('router:buildGraph', ...`` has a list of pipes that we need
    to add to:

    .. code:: diff

        gulp.task('router:buildGraph', gulp.series('router:copy', function () {
          gulp.src(['otp-data-container/*', 'otp-data-container/.*'])
            .pipe(gulp.dest(`${config.dataDir}/build/waltti`))
            .pipe(gulp.dest(`${config.dataDir}/build/finland`))
            .pipe(gulp.dest(`${config.dataDir}/build/hsl`))
            .pipe(gulp.dest(`${config.dataDir}/build/ulm`))

    .. todo:: provide patch for SKIP\_SEED

    Apply this patch, to support skipping the seed-step hsl is using to keep rebuilding the
    otp-data-container periodically. In our case, a fresh setup starting
    without an old container we could seed from, this sadly breaks every
    time.

    Apply by executing
    ``curl https://github.com/verschwoerhaus/OpenTripPlanner-data-container/commit/d657285fd2f73f11bedb9478be6880607b5b9733.patch | git apply``

    And now, we can finally build our own ``opentripplanner-data-container``!

    - Run ``npm install``
    - Run ``ROUTERS=ulm ORG=verschwoerhaus SKIP_SEED=true node index.js once``
      (Set ``ROUTERS=`` to your config identifier, set ``ORG`` to your docker
      hub username or organization)
    - Note the opentripplanner version the graph gets built with and save
      this information for later use. You can see this in the testing step
      of the build in a line like this:

    ::

        22:42:55.917 INFO (Graph.java:752) OTP version:   MavenVersion(1, 5, 0, SNAPSHOT, da7ca2a4d5a8cb381cd64efc6df5ba4252d45440)

    This OTP version is also the version of otp that has to run to ingest
    the data container again - and is needed for the container image tag of
    otp below when building the kubernetes config.

    After running the command (this could take a few minutes), you should
    see a new image appear in ``docker images``:

    ::

        REPOSITORY                                          TAG                                        IMAGE ID            CREATED             SIZE
        hsldevcom/opentripplanner-data-container-ulm        test                                       9742c641ad50        2 minutes ago      209MB

    You can now retag this image with your docker hub organization and
    correct tag and push it to docker hub:

    ::

        docker tag hsldevcom/opentripplanner-data-container-ulm:test verschwoerhaus/opentripplanner-data-container-ulm:2020-01-21
        docker push verschwoerhaus/opentripplanner-data-container-ulm:2020-01-21

Method 2: muenster-style custom container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`Code for Münster <https://codeformuenster.org/>`__ inspired us to use a simpler building
process by introducing a custom dockerfile for the datacontainer.

For this, we're going to fork the `digitransit-otp-data repository <https://github.com/verschwoerhaus/digitransit-otp-data>`__.

The ``Dockerfile`` is the main file you have to edit.
Below ``# add build data`` you see a list of ``ADD`` statements. Replace these URLs with those of your GTFS and OSM dump(s) (in the pbf format).
For the packaging, define your own ``ROUTER_NAME`` in the line ``ENV ROUTER_NAME=...``.

You can modify more graph bulding settings in the ``build-config.json``. The OpenTripPlanner Documentation contains a section about
`Graph build configuration <https://docs.opentripplanner.org/en/latest/Configuration/#graph-build-configuration>`__, listing a lot of settings and their default values.
For the ``router-config.json`` there also exists `Documentation with description <https://docs.opentripplanner.org/en/latest/Configuration/#runtime-router-configuration>`__ of the options and their default values.

If you have an GBFS feed, you can add an otp updater config to ``router-config.json`` like this:

::

    ...
    "updaters": [
      {
        "id": "openbike-bike-rental",
        "type": "bike-rental",
        "sourceType": "gbfs",
        "url": "https://api.openbike.ulm.dev/gbfs/",
        "frequencySec": 10,
        "network": "openbike"
      }
    ]
    ...

The building of the graph happens with the `mfdz OpenTripPlanner fork <https://github.com/mfdz/opentripplanner>`__.
It is important that the OpenTripPlanner version that builds the graph is the same that later serves the graph. If you want to update, get the latest docker image tag from the
`docker hub page of mfdz/opentripplanner <https://hub.docker.com/r/mfdz/opentripplanner/tags>`__ and modify ``OTP_VERSION`` in the Dockerfile.

For building and publishing, standard docker commands are used:

::

    docker build -t verschwoerhaus/opentripplanner-data-container-ulm:2020-01-21 .
    docker push verschwoerhaus/opentripplanner-data-container-ulm:2020-01-21


2. Building hsl-map-server
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. hint:: 
   We are using :term:`hsl-map-server` only for the stop (and bike)
   overlays. The basemap *can* be rendered by this project, but so far
   we have instead been using other tile servers. 
   In the beginning, we had used `Wikimedia's tile server <https://foundation.wikimedia.org/wiki/Maps_Terms_of_Use>`_,
   but a subsequent rush of third-party users during spring of 2020
   brought that service to it's knees, and `now its gone <https://lists.wikimedia.org/pipermail/maps-l/2020-August/001729.html>`__.
   You may configure your own (either homemade or factory-bought) tile
   server in digitransit-ui instead.

Check out `HSLdevcom/hsl-map-server <https://github.com/HSLdevcom/hsl-map-server>`__: ``git clone https://github.com/HSLdevcom/hsl-map-server``

Edit ``config.js``, modify ``module.exports`` to keep only the
``stop-map`` (rename ``hsl-stop-map`` into ``stop-map`` for this) and the ``citybike-map`` (only if needed) map layer:

::

    module.exports = {
      "/map/v1/stop-map": {
        "source": `otpstops://${process.env.OTP_URL}`,
        "headers": {
          "Cache-Control": "public,max-age=43200"
        }
      },
      "/map/v1/citybike-map": {
        "source": `otpcitybikes://${process.env.OTP_URL}`,
        "headers": {
          "Cache-Control": "public,max-age=43200"
        }
      },
    }

To build, run ``docker build -t verschwoerhaus/hsl-map-server:2020-01-21 .``

Push the resulting image also into docker hub:``docker push verschwoerhaus/hsl-map-server:2020-01-21``

3. Using photon-pelias-adapter
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

digitransit originally uses :term:`pelias` as a geocoder: Insert address,
get geocoordinates as a result. 
Sadly, pelias is not maintained
anymore - and custom adjustments seem to be very hard. We've therefore
decided to use `photon with an
adapter <https://github.com/stadtulm/photon-pelias-adapter>`_ instead.
(Photon has also problems, especially currently not supporting :term:`GTFS` stop
imports, but this should be solveable in the long run)

The adapter is completely configurable with one ENV variable
``PHOTON_URL``. It doesn't need to be custom built.

Later, we're simply using the docker container
`stadtulm/photon-pelias-adapter from docker
hub <https://hub.docker.com/r/stadtulm/photon-pelias-adapter>`_.

4. Building digitransit-ui
~~~~~~~~~~~~~~~~~~~~~~~~~~

Check out `HSLdevcom/digitransit-ui <https://github.com/HSLdevcom/digitransit-ui>`__: ``git clone https://github.com/HSLdevcom/digitransit-ui``

To build your own digitransit user interface, you need to add a theme
and provide configuration (which includes your custom urls).

First run ``yarn install``

To create the theme files, run ``yarn run add-theme <name>`` (you could
optionally supply a color and logo, read
`documentation <https://github.com/HSLdevcom/digitransit-ui/blob/master/docs/Themes.md>`__
for more details)

In ``app/configurations/``, a config file is created with your theme name, e.g. ``config.ulm.js``.

Replace this file with the contents of 
https://raw.githubusercontent.com/verschwoerhaus/digitransit-ui/ulm/app/configurations/config.vsh.js

For the configuration options, feel free to have a look into all the other files, preferential
``config.hsl.js``, ``waltti.js``, ``config.matka.js`` and ``config.default.js``.

The most basic configuration options you may want to change follow:

::
    
    const APP_TITLE = 'digitransit ';
    const APP_DESCRIPTION = 'digitransit - ber';


Define the bounding box of the area in which search queries are preferred.
Use a tool like https://boundingbox.klokantech.com to draw a bounding box
and fill the following constants:

::

    const minLat = 60;
    const maxLat = 70;
    const minLon = 20;
    const maxLon = 31;

You have to provide your own urls and paths with your config name, eg.
in

::

    OTP: process.env.OTP_URL || `${API_URL}/routing/v1/routers/ulm/`,
    // ...
    STOP_MAP: `${API_URL}/map/v1/stop-map/`,

Enter your used :term:`GTFS` feed ids in

::

    feedIds: ['DING'],

Configure the tile server of your choice. For testing, you might be content
using the german osm tile server:

::

    const MAP_URL = 'https://{s}.tile.openstreetmap.de/';

and inside the config part:

::

    map: {
        useRetinaTiles: true,
        tileSize: 256,
        zoomOffset: 0,
    },

You also have to supply your own ``themeMap``, so your theme gets
recognized and used:

::

    themeMap: {
        ulm: 'ulm',
    },

For more config options that we set, have a look into
https://github.com/verschwoerhaus/digitransit-ui/blob/ulm/app/configurations/config.vsh.js

Finally, also create an docker image out of the ui: ``docker build -t verschwoerhaus/digitransit-ui:2020-01-21 .``
Push the resulting image to docker hub: ``docker push verschwoerhaus/digitransit-ui:2020-01-21``

5. Building digitransit-proxy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. warning:: 
    digitransit-proxy is deprecated as of 2020-08 and is only
    preserved for archival reasons. We're using the k8s ingress
    now, thats introduced in the next step

.. container:: toggle

    .. todo::
        digitransit-proxy can be completely replaced by
        kubernetes-ingress-nginx. see cfm:
        https://github.com/codeformuenster/kubernetes-deployment/blob/46ea8118ff55fb2d3158d61a96e6d92ac3b951ee/sources/digitransit/ingress.yaml

    nginx will not start if it cannot resolve the hostnames in its (proxy)
    configuration. This is why we have to fork the digitransit proxy and
    remove all the references to stuff we don't need.

    See the diff at
    https://github.com/HSLdevcom/digitransit-proxy/compare/master...transportkollektiv:master
    for all the location config you should remove.

    Note that some endpoints need your configuration name in the url, eg
    ``/routing/v1/routers/hsl`` → ``/routing/v1/routers/ulm``.

6. Crafting kubernetes yaml
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You need:

-  access to a kubernetes cluster
-  kubectl on your device,
   with `kubeconfig <https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/>`__
   for this cluster
-  already configured ingress in your kubernetes setup, for example with the `nginx ingress controller <https://kubernetes.github.io/ingress-nginx/>`__
-  url of your pelias service

We're going to connect the different parts to each other: 

- opentripplanner-data-container → opentripplanner (otp gets its graph from the data container)
- opentripplanner → hsl-map-server (mapserver gets its stop data from otp)
- photon → photon-pelias-adapter (its simply the URL of your photon)

Make some parts accessible from the outside:

- digitransit-ui → ingress
- hsl-map-server → ingress
- opentripplanner → ingress 
- opentripplanner-data-container → ingress (for debugging)
- photon-pelias-adapter → ingress

And then, digitransit-ui running in your browser can access these:

- hsl-map-server → digitransit-ui (public, by ingress url)
- opentripplanner → digitransit-ui (public, by ingress url)
- photon-pelias-adapter → digitransit-ui (public, by ingress url)

For all of this, we are building deployments and services for each of the containers.
Note that you have to use the right container image tags.

.. hint::
   Reminder: the OpenTripPlanner version has to match the version that was used to build the graph.
   Ensure you are using the same docker image tag here.

Have a look at this working template:
https://github.com/verschwoerhaus/digitransit-kubernetes/blob/master/all.yml

Edit the deployment container specs, modify the ``image`` and ``env`` keys.
For the environment variables, have a look at these of digitransit-ui (``CONFIG``), 
opentripplanner (``ROUTER_NAME``), and photon-pelias-adapter (``PHOTON_URL``).
The digitransit-ui container also needs the public urls to OTP (``OTP_URL``) and 
photon-pelias-adapter (``GEOCODING_BASE_URL``).

For these urls and to write the services up to the ingress, have a look at 
https://github.com/verschwoerhaus/digitransit-kubernetes/blob/master/ingress.yaml

For the parts you have to edit, look at the hostnames (``host:``) and at the paths (``path:``).

For handling HTTPS, add tls-keys like in this config:
https://github.com/stadtulm/digitransit-k8s/blob/master/ingress.yaml


.. todo::
   maybe provide existing template and only teach how to
   override/insert config with kustomize

7. Deploying
~~~~~~~~~~~~

Execute ``kubectl apply -f digitransit.yml`` and ``kubectl apply -f ingress.yml``

8. ???
~~~~~~

Watch ``kubectl get pods``.

Look at ``kubectl get ingress``

9. Profit!
~~~~~~~~~~

Access your digitransit instance. Test one route. Test more routes. Look
for edge cases. Have a little “test suite” prepared with standard trips
to check against. Do a little dance :)

TODO
~~~~

-  try with a "real" kubernetes cluster, not only single node. eg. GKE
-  bring upstream:

  -  https://github.com/verschwoerhaus/tilelive-otp-stops/commit/858e8fc7db5fbd206019236816a029259cf40582
