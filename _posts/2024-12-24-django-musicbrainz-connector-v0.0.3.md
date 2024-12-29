---
layout: post
title:  "Django MusicBrainz Connector v0.0.3"
---

**Django MusicBrainz Connector** is a Django app that connects to replica of the MusicBrainz database. In this third
release, we added models `Language`, `Medium Format`, `ReleaseGroupPrimaryType`, `ReleasePackaging`, `ReleaseStatus`,
`Script`, and `Track`, in addition to the already existing `ArtistCredit`, `LinkType`, `Link`, `Recording`,
`RecordingWorkLink`, `Work`, and `WorkType`.

We've also dropped support for the end-of-life Python 3.8 and added support for Python 3.11 and 3.12.

You can find the [package on PyPI](https://pypi.org/project/django-musicbrainz-connector/), the
[code on GitHub](https://github.com/mneia-gr/django-musicbrainz-connector), and the
[documentation on Read The Docs](https://django-musicbrainz-connector.readthedocs.io/en/latest/).
