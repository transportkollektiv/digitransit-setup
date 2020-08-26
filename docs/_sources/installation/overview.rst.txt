:github_url:  https://github.com/transportkollektiv/digitransit-setup/tree/main/src/installation/overview.rst

Overview
========

digitransit is a slightly complex system composed of multiple services
in container that talk to each other. A nice architecture image can be
found at the `digitransit.fi
documentation <https://digitransit.fi/en/developers/architecture/>`__.

The main components of an digitransit deployment consists of:

-  Multimodal routing engine (:term:`OpenTripPlanner`)
-  Address search (originally :term:`pelias`)
-  Background map service (:term:`hsl-map-server`: [tessera], [tilelive])
-  Web browser-based user interface (:term:`digitransit-ui`)

A nicer description of the components and their job exists also at
https://digitransit.fi/en/services/


.. blockdiag::

   diagram {

     tileserver [label="OSM Tile Server"];
     datacontainer [label="OTP Data Container"];
     gtfs [label="GTFS", stacked];
     gbfs [label="GBFS", stacked];
     osmgraph [label="OSM Street Graph"];
     otp [label="OpenTripPlanner"];
     hslmap [label="HSL Map Server"];
     pelias [label="Pelias"];
     tessera [label="tessera", style="dashed"];
     tilelive [label="tilelive", style="dashed"];
     dui [label="digitransit-ui"];

     tileserver, pelias -> dui;
     gtfs, gbfs, osmgraph -> datacontainer -> otp -> dui;
     otp, tessera, tilelive -> hslmap -> dui;

     otp -> hslmap [folded];
   }

.. note:: 
     As can be seen in the diagram, some dependencies are circular. This poses a few challenges when first setting up a working instance, which we will address later on.
