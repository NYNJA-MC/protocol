NYNJA PROTOCOL SPECIFICATION
============================

Version 1.0 Maxim Sokhatsky

OVERVIEW
--------

To support the development of messaging apps and IOT infrastructure,
NYNJA Protocol is designed from the ground up to be the most efficient,
stable and scalable protocol available.  NYNJA itself is an open-source
specification for Messaging and IOT applications and is a part of an
open source family of N2O protocols. While it can be used with any
transport we promote MQTT as an efficient binary protocol dedicated
for applications over unreliable networks such as GSM, CDMA and other
wireless networks.

NYNJA Protocol enables all companies, regardless of size, to build
systems that can compete with the world’s most sophisticated
communication platforms. NYNJA Protocol has been made open source
for the benefit of the global community of developers. NYNJA Protocol
powers the NYNJA Mobile Communicator.

FEATURES
--------

* Versioning
* Two-topic: Actions/Events Async Event Bus Topics
* Media: Voice, Video, File Transfer, GPS, Text
* Multi-User Conferences (Tribes)
* Multi Accounts (Rosters)

INTRO
-----

* Messages: Schema Definition
* Endpoints: Topic Subscriptions (MQTT), WebSockets (WS)

URI Components
--------------

### Client Subscriptions:

* `actions/1/index/emqttd_198234215548221` &mdash; version 1
* `actions/2/index/emqttd_198234215548221` &mdash; version 2
* `actions/1/api/client/token` &mdash; version 2

### Server Subscriptions:

* `events/1/api/anon/client/token` &mdash; version 2
* `events/2/3/index/maxim@synrc.com/emqttd_198234215548221`
* `events/1/2/login/anon/emqttd_198234215548221`

### Review Application Subscriptions:

* `room/n2o`

Metawords
---------

* `randomly` — may be received by client randomly
* `once` — should be received once according to sub-protocol section
* `issues` — may be send randomly.
* `sends` — ususal sending
* `participants` — depending on number of mates

Sub Protocols
-------------

* [INIT](https://github.com/NYNJA-MC/protocol/blob/master/INIT.md) — 1
* [FTP](https://github.com/NYNJA-MC/protocol/blob/master/FTP.md) — 1
* [AUTH](https://github.com/NYNJA-MC/protocol/blob/master/AUTH.md) — 3
* [MESSAGE](https://github.com/NYNJA-MC/protocol/blob/master/MESSAGE.md) — 3
* [PUBLIC](https://github.com/NYNJA-MC/protocol/blob/master/PUBLIC.md) — 6
* [PRIVATE](https://github.com/NYNJA-MC/protocol/blob/master/PRIVATE.md) — 4
* [ROSTER](https://github.com/NYNJA-MC/protocol/blob/master/ROSTER.md) — 7
* [PROFILE](https://github.com/NYNJA-MC/protocol/blob/master/PROFILE.md) — 3

Payloads
--------

Payloads are MIME-driven, so Video, Audio and Text messages correspond to MIME types.
**Events** come from server and is a subscription topic for client,
**Actions** come from client and is a publishing topic for client.
This is a front channel for raw BERT messages.
Other topics may bind to event or action topics.

Credits
-------

* Marshall Taplits
* Maxim Sokhatsky
* Yuri Maslovsky
* Igor Kharin
* Liubov Mykhailova
