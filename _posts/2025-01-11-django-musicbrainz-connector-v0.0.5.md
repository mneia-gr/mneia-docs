---
layout: post
title: "Django MusicBrainz Connector v0.0.5"
---

**Django MusicBrainz Connector** is a Django app that connects to replica of the MusicBrainz database. Release v0.0.5 is
a bug fix release. Changes:

*   This package requires a dependency on `psycopg` to be able to connect to the PostgreSQL database, but the dependency
    was not declared.
*   All URLs had the string `/api` hardcoded as a prefix. This was removed, so that the user can decide the URL
    structure.
*   Some fields on models `Area`, `AreaType`, and `Artist` are optional in their SQL definition, but they were not
    flagged as optional in the Django MusicBrainz Connector.
*   Some fields on models `Area`, `AreaType` and `Artist` were missing the `db_column` attribute, so they were not able
    to retrieve data from the database.
