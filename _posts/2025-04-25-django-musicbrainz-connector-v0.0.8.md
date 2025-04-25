---
layout: post
title: "Django MusicBrainz Connector v0.0.8"
---

**Django MusicBrainz Connector** is a Django app that connects to replica of the MusicBrainz database.

In version v0.0.8 we added an ability in the API to GET an Artist using either the Artist's `id` (an integer), or `gid`
(a UUID), or the Artist's name (as long as the name is unique). Either of these three calls should return the same
result:

```
GET /api/artists/738375/
GET /api/artists/f61458a1-412e-46c0-ad76-d2e0c39a14ff/
GET /api/artists/Κώστας Ουράνης/
```
