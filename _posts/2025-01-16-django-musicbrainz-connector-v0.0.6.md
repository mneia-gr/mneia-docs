---
layout: post
title: "Django MusicBrainz Connector v0.0.6"
---

**Django MusicBrainz Connector** is a Django app that connects to replica of the MusicBrainz database. Release v0.0.6 is
a bug fix release. Changes:

*   Added missing `db_columb` attribute to the `Link` model.
*   Added missing `default` value to the `LinkType` model.
*   Removed the `priority` field from the `LinkType` model, per the same upstream change.
*   Dropped support for Django 4 and require Django 5.1.
*   Updated versions of required dependencies.
