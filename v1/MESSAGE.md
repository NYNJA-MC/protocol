MESSAGE: Sending and Receiving Messages
=======================================

Version 1.0 Maxim Sokhatsky

Endpoints
--------

* `p2p/:phone_id/:phone_id` — MQTT
* `actions/1/api/phone/:phone` — MQTT
* `events/1//api/anon/:client/:token` — MQTT

Tuples
------

```erlang
-record('Typing',   {author=[] :: list(),
                     room=[] :: list()}).
```

```erlang
-record(muc,       {name = [] :: [] | binary() }).
-record(group,      {name = [] :: [] | binary() }).
-record(p2p,        {from = [] :: [] | binary(),
                     to   = [] :: [] | binary() }).

```

```erlang
-record('History',  {roster_id= [] :: [] | binary(),
                     contact_id=[] :: [] | binary(),
                     cursor :: [] | integer() | binary(),
                     data=[] :: list(#'Message'{}),
                     status=[] :: get | updated
                             | last_loaded | last_msg}).
```

```erlang
-record('Message',  {id=[] :: [] | integer(),
                     container = chain | cur,
                     feed_id=[] :: #p2p{} | #muc{}},
                     prev=[] :: [] | integer(),
                     next=[] :: [] | integer(),
                     msg_id = [] :: [] | binary(),
                     from = [] :: [] | binary(),
                     to = [] :: [] | binary(),
                     created = [] :: [] | integer() | binary(),
                     files = [] :: [] | list(#'Desc'{}),
                     type = [] :: [] | reply | forward | sched
                          | online | offline | join | leave
                          | removed,
                     edit_msg = [] :: [] | integer(),
                     status = [] :: [] | atom() | client | async
                            | sent | internal | last_read | edit }).
```

```erlang
-record('Desc',     {mime    = [] :: [] | binary(),
                     payload = [] :: [] | binary(),
                     filename= [] :: [] | binary(),
                     info= [] :: [] | binary()}).
```

```erlang
-record('Cursor',   {phone    = [] :: {p2p, binary(), binary()},
                     roster   = [] :: integer(),
                     position = [] :: integer()}).

```

Overview
--------

MESSAGE API deliver messages.

Protocol
--------

### `Message/client` — General Sending Message to Subscribers

```
1. client sends `{'Message',[],_,Feed,_,_,Id,From,To,Files,Time,Type,_,client}`
             to `events/1//api/anon/:client/:token` once.
```

Examples:

* From — `380676631870_1`
* To — `380676631870_1` or `lobby`
* Feed — `{p2p,_,_}` or `{muc,_}`

```
2. server sends `{'Message',Id,_,_,_,_,_,To,From,Files,Time,Type,_,sent}`
             to `p2p/:address` twice
             or `room/:room` members times.
```

### `Message/upload` — Sending Async Upload Message

```
1. client sends `{'Message',[],_,_,_,_,Id,From,To,Files,Time,Type,_,upload}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{'Message',Id,_,_,_,_,_,To,From,Files,Time,Type,_,link}`
             to `p2p/:address' twice
             or `room/:to` members times.
```

```
3. client sends `{'Message',Id,_,_,_,_,_,From,To,Files,Time,Type,_,complete}`
             to `events/1//api/anon/:client/:token` once.
```

```
4. server sends `{'Message',Id,_,_,_,_,_,To,From,Files,Time,Type,_,sent}`
             to `p2p/:address' twice
             or `room/:to` members times.
```

### `Message/edit` — Edit/Remove Message

```
1. client sends `{'Message',Id,_,_,_,_,_,_,_,Files,Time,Type,_,edit}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{'Message',Id,_,_,_,_,_,_,_,Files,Time,Type,_,edit}`
             to `p2p/:address' twice
             or `room/:to` member times.
```

### `History/get` — Retrieve History

```
1. client sends `{'History',Id,Contact,MsgId,_,get}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{'History',Id,Contact,MsgId,Messages,get}`
             to `actions/1/api/:client` once or more.
```

### `Cursor/` — Set cursor

Sets marker to feed for counterparty.

```
1. client sends `{'Cursor',Feed,Side,Position}`
             to `events/1//api/anon/:client/:token` once.
```

* Feed — `#p2p{}` key of the conversation
* Side — `1` or `2` for counterparty
* Position — id of the message you want cursor to point

```
2. server sends `{ok2,Feed,Count}`
             to `actions/1/api/:client` once or more.
```
