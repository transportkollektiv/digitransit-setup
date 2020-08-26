:github_url:  https://github.com/transportkollektiv/digitransit-setup/tree/main/src/installation/faq.rst

Frequently Asked Questions
==========================

-  **I see no frequently asked questions?!** – Feel free to ask one :)
-  **I did everything as you say, but when I test bus relations, the route
   is all zigzagging over the map instead of following the road**
   –  Your :term:`GTFS` feed is missing ``shapes.txt``. This happens 
   occasionally, depending on where you get your feed from. See `this blog
   post <https://ulm.dev/2020/01/17/pfaedle/>`_ on how to integrate
   them into your feed yourself