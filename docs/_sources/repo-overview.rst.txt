:github_url:  https://github.com/transportkollektiv/digitransit-setup/tree/main/src/repo-overview.rst

Repository Overview
===================

If you want to look into different digitransit instances, which features they support, a look into the repositories is always a good idea.
Sometimes you can reverse engineer the used parts by looking at the deployment files. For an easier overview, we've listed the parts below.
If something changes or you also want to list a digitransit instance here, feel free to edit these docs. 

Herrenberg, stadtnavi
---------------------
Improved digitransit with lots of layers and features for Herrenberg: https://herrenberg.stadtnavi.de

* UI: https://github.com/stadtnavi/digitransit-ui

* OpenTripPlanner Data: https://github.com/mfdz/OpenTripPlanner-data-container

* OpenTripPlanner: https://github.com/mfdz/OpenTripPlanner

* Map Server: https://github.com/stadtnavi/hsl-map-server
  
  * roadworks https://github.com/stadtnavi/tilelive-roadworks-bw
  
  * parkapi https://github.com/stadtnavi/tilelive-park-api
  
  * stops https://github.com/stadtnavi/tilelive-otp-stops

  * citybikes https://github.com/stadtnavi/tilelive-otp-citybikes

* Pelias: using photon instead, https://github.com/stadtulm/photon-pelias-adapter

* Deployment: ansible, https://github.com/stadtnavi/digitransit-ansible

HSL
---
*The original one*. Available at https://reittiopas.hsl.fi

* UI: https://github.com/HSLdevcom/digitransit-ui

* OpenTripPlanner Data: https://github.com/HSLdevcom/OpenTripPlanner-data-container

* OpenTripPlanner: https://github.com/HSLdevcom/OpenTripPlanner

* Map Server: https://github.com/HSLdevcom/hsl-map-server

* Graphiql: https://github.com/HSLdevcom/graphiql-deployment

* Pelias: https://github.com/HSLdevcom/pelias-data-container

* Proxy: https://github.com/HSLdevcom/digitransit-proxy

* Deployment: azure aks/k8s, https://github.com/HSLdevcom/digitransit-kubernetes-deploy

Oulu
----
Interesting UI changes, lots of additional information like traffic cams and snow plowers. https://www.oulunliikenne.fi

* UI: https://github.com/digiaonline/oulunliikenne-digitransit-ui

For the other services, the UI uses the ones provided by HSL.

Turin
-----
Available at [muoversiatorino.it](https://www.muoversiatorino.it/-/), developed by [5t](http://www.5t.torino.it/en/muoversi-a-torino/)

* UI: https://github.com/5Tsrl/digitransit-ui

* OpenTripPlanner: https://github.com/5Tsrl/OpenTripPlanner

* OpenTripPlanner Data: ???

* Map Server: https://github.com/5Tsrl/hsl-map-server
	
	* citybikes: https://github.com/5Tsrl/tilelive-otp-citybikes

* Deployment: ???

Ulm, Verschwörhaus
------------------
The staging, test and development instance. Public at https://digitransit.im.verschwoerhaus.de

* UI: https://github.com/verschwoerhaus/digitransit-ui/tree/ulm -- branch: ulm, config: ``config.vsh.js``

* OpenTripPlanner Data: https://github.com/verschwoerhaus/digitransit-otp-data

* OpenTripPlanner: using mfdz/opentripplanner

* Map Server: https://github.com/verschwoerhaus/hsl-map-server
  
  * stops https://github.com/verschwoerhaus/tilelive-otp-stops
  
  * citybikes https://github.com/verschwoerhaus/tilelive-otp-citybikes

* Graphiql: https://github.com/verschwoerhaus/graphiql-deployment/tree/vsh -- branch: vsh

* Pelias: using photon instead, https://github.com/stadtulm/photon-pelias-adapter

* Deployment: k8s, https://github.com/verschwoerhaus/digitransit-kubernetes


Ulm, ulm.dev
------------
The more productive instance. Public at https://digitransit.ulm.dev

* UI: https://github.com/verschwoerhaus/digitransit-ui/tree/ulm -- branch: ulm, config: ``config.ulm.js``

* OpenTripPlanner Data: https://github.com/stadtulm/digitransit-otp-data

* OpenTripPlanner: using mfdz/opentripplanner

* Map Server: https://github.com/verschwoerhaus/hsl-map-server
  
  * stops https://github.com/verschwoerhaus/tilelive-otp-stops

  * citybikes https://github.com/verschwoerhaus/tilelive-otp-citybikes

* Graphiql: https://github.com/verschwoerhaus/graphiql-deployment/tree/vsh -- branch: vsh

* Pelias: using photon instead, https://github.com/stadtulm/photon-pelias-adapter

* Deployment: k8s, https://github.com/stadtulm/digitransit-k8s
