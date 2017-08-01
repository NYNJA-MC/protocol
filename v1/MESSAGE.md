MESSAGE: Sending and Receiving Messages
=======================================

Version 1.0 Maxim Sokhatsky

Endpoints
--------

* `actions/1/api/phone/:phone` — MQTT
* `p2p/:from_phone_id/:to_phone_id` — MQTT
* `p2p/:from_phone_id` — MQTT
* `p2p/:from_phone_id/to_phone_id/:client` — MQTT
* `events/1//api/anon/:client/:token` — MQTT

Tuples
------

```erlang
-record('Typing',   {author=[] :: list(),
                     room=[] :: list()}).
```

```erlang
-record('History',  {roster_id= [] :: [] | binary(),
                     contact_id=[] :: list(),
                     last_loaded_msg_id :: [] | integer() | binary(),
                     data=[] :: list(#'Message'{}),
                     status=[] :: atom() | updated | last_loaded | last_msg}).
```

```erlang
-record('Message',  {id=[] :: [] | integer(),
                     container=Container :: atom(),
                     feed_id=[] :: term() | {p2p, binary(), binary()},
                     prev=[] :: [] | integer(),
                     next=[] :: [] | integer(),
                     feeds=[] :: list(),
                     msg_id = [] :: [] | binary(),
                     from = [] :: [] | binary(),
                     to = [] :: [] | binary(),
                     sync = [] :: [],
                     created = [] :: [] | integer() | binary(),
                     access = [] :: [],
                     starred = [] :: [],
                     payload = <<>> :: [] | binary(),
                     mime = [],
                     seen_by = [],
                     edit_msg = [] :: [] | integer(),
                     status = [] :: [] | atom() | client | sent | offline| online | internal | last_read | edit}).
```
Overview
--------

MESSAGE API deliver messages.

Protocol
--------

### Sending P2P Message


```
1. client sends `{'Message',_,_,_,_,_,_,_,FromPhoneId,ToPhoneId,_,_,_,_,Payload,_,_,client}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{'Message',Id,_,_,_,_,_,_,FromPhoneId,ToPhoneId,_,Created,_,_,Payload,_,_,sent}`
                to `p2p/:from_phone_id/:to_phone_id` counterparty.
             sends `{'Message',Id,_,_,_,_,_,_,FromPhoneId,ToPhoneId,_,Created,_,_,Payload,_,_,sent}`
                to `actions/1/api/phone/:from_phone` issuer to all `from` sessions.
             sends `{'Roster',FromId,_,_,_,[{Contact,FromPhone+Id,_,_,_,_,Unread=0,LastMsg,_,last_msg}],_,_,Phone,_,_,last_msg}` where LastMsg - last sent message
                to `p2p/:to_phone_id/:from_phone_id` as retain.
             sends `{'Roster',ToId,_,_,_,[{Contact,ToPhone+Id,_,_,_,_,Unread+1,LastMsg,_,last_msg}],_,_,Phone,_,_,last_msg}` where LastMsg - last sent message
                to `p2p/:from_phone_id/:to_phone_id` as retain.

```

### P2P Read/Unread
```
1. client sends `{'Message',LastReadId,_,_,_,_,_,_,FromPhoneId,ToPhoneId,_,_,_,_,_,_,_,last_read}` drops unread_counter
             to `events/1//api/anon/:client/:token` once and marks message as read.

2. server sends `{'Roster',FromId,_,_,_,[{Contact,FromPhone+Id,_,_,_,_,0=Unread,LastMsg,_,last_msg}],_,_,Phone,_,_,last_msg}`
                to `p2p/:to_phone_id/:from_phone_id` as retain.

```

### P2P Edit/Remove Message

```
1. client sends `{'Message',Id,_,_,_,_,_,_,FromPhoneId,ToPhoneId,_,_,_,_,Payload,SeenBy,EditMsgId,edit}`
            SeenBy is `-1` if message removed or `0` if seen by all or `1` if seen by only `from` user or `2` if seen by only `to` user
             to `events/1//api/anon/:client/:token` once and marks message as read.

2. server sends `{'Message',Id,_,_,_,_,_,_,FromPhoneId,ToPhoneId,_,Created,_,_,Payload,SeenBy,EditMsgId,edit}`
                to `p2p/:from_phone_id/:to_phone_id` counterparty.
             sends `{'Message',Id,_,_,_,_,_,_,FromPhoneId,ToPhoneId,_,Created,_,_,Payload,SeenBy,EditMsgId,edit}`
                to `actions/1/api/phone/:from_phone` issuer to all "from" sessions.
```


### Retrieve History

```
1. client sends `{'History',Id,Contact,LastLoadedMsgId,_,update}`
            where LastLoadedMsgId is message from where history has been loaded for the session.
             If LastLoadedMsgId is empty or 0 then the all history must be pulled for the session
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{'History',Id,Contact,NewLastLoadedMsgId,Messages,update}`
        where NewLastLoadedMsgId is the new last retreived message id for the session.
          Messages are exectly from LastLoadedMsgId to NewLastLoadedMsgId
             to `actions/1/api/:client` once or more.
```

### Presence Messages

```
1. server sends `{'Message',_,_,_,_,_,_,_,PhoneId,_,_,Created,_,_,_,_,_,Status}`
        where Status is `online` by login or `offline` by disconnect
             to `p2p/:phone_id`
```
