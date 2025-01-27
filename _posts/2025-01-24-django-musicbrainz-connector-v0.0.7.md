---
layout: post
title: "Django MusicBrainz Connector v0.0.7"
---

**Django MusicBrainz Connector** is a Django app that connects to replica of the MusicBrainz database. Changes in
release v0.0.7:

*   Added models `LinkAttribute`, `LinkAttributeTextValue`, `LinkAttributeType`, and `LinkTextAttributeType`.
*   Added some missing `db_column` field attributes on model `LinkType`, and added a test so that these aren't missed.
*   Added complete or partial data dumps from the MusicBrainz database into the Sphinx documentation of models
    `AreaType`, `ArtistType`, `Gender`, `LinkAttributeType`, and `WorkType`.
