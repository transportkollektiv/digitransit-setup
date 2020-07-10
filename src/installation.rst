Installation guide
==================

You've seen https://digitransit.fi and some instances like
https://digitransit.im.verschwoerhaus.de https://mobil-in-herrenberg.de
https://digitransit.codeformuenster.org https://cityrouting.e-gpp.hr
https://next-dev.oulunliikenne.fi. Now you're interested in how to get a
digitransit for your own region. This could be a helpful step-by-step
guide for you :)

Ingredients
-----------

-  One host machine with a-little-bit-more-than-average RAM (i.e. 16+ GB) for
   building
-  One Kubernetes Cluster (can be single-node) with 8+ GB RAM
-  `kubectl <https://kubernetes.io/docs/tasks/tools/install-kubectl/>`__
   installed and configured on your device
-  One OpenStreetMap extract for your region (see:
   https://download.geofabrik.de)
-  One (or more) :term:`GTFS` Feeds for public transit routes
-  A GitHub Account (for forking stuff)
-  A Docker Hub Account (for publishing your containers)
-  One Domain and access to DNS for creating a subdomain

optional
~~~~~~~~

-  One (or more) GBFS Feeds (for bikesharing and/or carsharing)

Overview
--------

digitransit is a slightly complex system composed of multiple services
in container that talk to each other. A nice architecture image can be
found at the `digitransit.fi
documentation <https://digitransit.fi/en/developers/architecture/>`__.

The main components of an digitransit deployment consists of:

-  Multimodal routing engine ([OpenTripPlanner])
-  Address search ([Pelias])
-  Background map service ([hsl-map-server]: [tessera], [tilelive])
-  Web browser-based user interface ([digitransit-ui])

A nicer description of the components and their job exists also at
https://digitransit.fi/en/services/

Step-by-Step
------------

0. Preparing your build host
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

you need:

- docker-ce runtime
- git
- nodejs, LTS version
- yarn

.. note:: This should be enhanced with copy-pasteable commands

1. Building an OpenTripPlanner Graph
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

you need: 

- One (or more) :term:`GTFS` Feed with a publicly accessible URL
  (if you only have a :term:`GTFS` zip file, upload it somewhere
  public)
- One OpenStreetMap extract for your region in .pbf format,
  accessible at a public URL - try https://download.geofabrik.de 
- if you need to merge two regions, try
  `osmium merge <https://gis.stackexchange.com/questions/242704/merging-osm-pbf-files>`__

For the configuration, find and memorize a short identifier. You
need this often. In our example, we use ``ulm``.

Method 1: vsh-style modifying of opentripplanner-data-container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

(remove stuff like ``"fares": "HSL",``, this is not relevant outside of
finland)

edit ``router-config.js`` too, the ``routingDefaults`` are mostly okay.
If you are not satisfied with the routing suggestions, try to modify
these values. ``updaters`` are for realtime updates to feeds (think:
GTFS-RT) or GBFS (Bikesharing, Carsharing) status updates. If you don't
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

Until PR #XX <> is merged, we have to apply this patch, to support
skipping the seed-step hsl is using to keep rebuilding the
otp-data-container periodically. In our case, a fresh setup starting
without an old container we could seed from, this sadly breaks every
time.

Apply by executing
``curl https://github.com/HSLdevcom/OpenTripPlanner-data-container/commit/d657285fd2f73f11bedb9478be6880607b5b9733.patch | git apply``

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

`Code for Münster <https://codeformuenster.org/>`__ used a simpler building
process by introducing custom dockerfiles for opentripplanner and the 
datacontainer.

.. todo::
  https://github.com/codeformuenster/OpenTripPlanner
  https://github.com/codeformuenster/OpenTripPlanner-graph

.. todo:: 
   edit cfm reference in the dockerfiles, build an own dockerfile
   containing your urls for otp-graph.

2. Building hsl-map-server
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note:: 
   We are using ``hsl-map-server`` only for the stop (and bike)
   overlays. The basemap *can* be rendered by this project, but we will 
   still be replacing that by the `wikimedia tile server <https://foundation.wikimedia.org/wiki/Maps_Terms_of_Use>`__,
   configured in digitransit-ui.

.. todo::
   disregard the last statement. We need a different map server now :D

Check out `HSLdevcom/hsl-map-server <https://github.com/HSLdevcom/hsl-map-server>`__: ``git clone https://github.com/HSLdevcom/hsl-map-server``

Edit ``config.js``, modify ``module.exports`` to keep only the
``stop-map`` (and the citybike, if needed) map layer:

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

3. Building digitransit-proxy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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

4. Using photon-pelias-adapter
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

digitransit originally uses pelias. Sadly, pelias is not maintained
anymore - and custom adjustments seem to be very hard. We've therefore
decided to use `photon with an
adapter <https://github.com/stadtulm/photon-pelias-adapter>`__ instead.
(Photon has also problems, especially currently not supporting :term:`GTFS` stop
imports, but this should be solvable in the long run)

The adapter is completely configurable with one ENV variable
``PHOTON_URL``. It doesn't need to be custom built.

Later, we're simply using the docker container
`stadtulm/photon-pelias-adapter from docker
hub <https://hub.docker.com/r/stadtulm/photon-pelias-adapter>`__.

5. Building digitransit-ui
~~~~~~~~~~~~~~~~~~~~~~~~~~

To build your own digitransit user interface, you need to add a theme
and provide configuration (which includes your custom urls).

First run ``yarn install``

To create the theme files, run ``yarn run add-theme <name>`` (you could
optionally supply a color and logo, read
`documentation <https://github.com/HSLdevcom/digitransit-ui/blob/master/docs/Themes.md>`__
for more details)

In ``app/configurations/``, create your own ``config.ulm.js``. For the
configuration content, look into all the other files, preferential
``config.hsl.js``, ``waltti.js``, ``config.matka.js`` and
``config.default.js``.

You have to provide your own urls and paths with your config name, eg.
in

::

    OTP: process.env.OTP_URL || `${API_URL}/routing/v1/routers/ulm/`,
    // ...
    STOP_MAP: `${API_URL}/map/v1/stop-map/`,

Enter your used :term:`GTFS` feed ids in

::

    feedIds: ['DING'],

For using the wikimedia tile server, use

::

    const MAP_URL = 'https://maps.wikimedia.org/osm-intl/';

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

6. Crafting kubernetes yaml
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You need:

-  access to a kubernetes cluster
-  kubectl on your device,
   with `kubeconfig <https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/>`__
   for this cluster
-  Build services for each of the containers and deployments with
   the right container image tags. (Especially important for opentripplanner)

Connect the different parts to each other: 

- digitransit-ui → digitransit-proxy 
- hsl-map-server → digitransit-proxy 
- opentripplanner → digitransit-proxy 
- opentripplanner-data-container → digitransit-proxy 
- opentripplanner-data-container → opentripplanner (otp gets its graph from the data container)
- opentripplanner → hsl-map-server (mapserver gets its stop data from otp)

Don't forget the environment variables for digitransit-ui (``CONFIG``)
and opentripplanner (``ROUTER_NAME``).

Have a look at this working template:
https://github.com/verschwoerhaus/digitransit-kubernetes/blob/master/all.yml

.. todo::  
   rewrite section, what services have to exist, how should they be
   named, how to get step by step to kubernetes config
.. todo::
   maybe provide existing template and only teach how to
   override/insert config with kustomize

7. Deploying
~~~~~~~~~~~~

Execute ``kubectl apply -f digitransit.yml``

8. ???
~~~~~~

Watch ``kubectl get pods``

9. Profit!
~~~~~~~~~~

Access your digitransit instance. Test one route. Test more routes. Look
for edge cases. Have a little “test suite” prepared with standard trips
to check against. Do a little dance :)

Frequently Asked Questions
--------------------------

-  I see no frequently asked questions?!
-  Feel free to ask one :)
-  I did everything as you say, but when I test bus relations, the route
   is all zigzagging over the map instead of following the road
-  Your :term:`GTFS` feed is missing ``shapes.txt``. This happens occasionally,
   depending on where you get your feed from. See `this blog
   post <https://ulm.dev/2020/01/17/pfaedle/>`__ on how to integrate
   them into your feed yourself

TODO
----

-  try with a "real" kubernetes cluster, not only single node. eg. GKE
-  bring upstream:
-  https://github.com/verschwoerhaus/tilelive-otp-stops/commit/858e8fc7db5fbd206019236816a029259cf40582
-  https://github.com/verschwoerhaus/OpenTripPlanner-data-container/commit/d657285fd2f73f11bedb9478be6880607b5b9733

