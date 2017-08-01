SEARCH: Directory Search
========================

Version 1.0 Maxim Sokhatsky

Endpoints
--------

* `actions/1/api/phone/:phone` — MQTT
* `actions/1/api/:client` — MQTT
* `events/1//api/anon/:client/:token` — MQTT

Tuples
------


```
-record('Search',   {id = [] :: [] | integer(),
                     phone = [] :: [] | binary(),
                     names = [] :: [] | binary(),
                     surnames = [] :: [] | binary(),
                     query = [] :: [] | binary(),
                     status = [] :: [] | contact | atom()}).
```

```erlang
-record('Roster',   {id=[] :: [] | integer(),
                     names=[] :: [] | binary(),
                     surnames=[] :: [] | binary(),
                     size=0 :: integer(),
                     userlist=[] :: list(#'Contact'{}),
                     roomlist=[] :: list(#'Room'{}),
                     subscribe=true :: boolean(),
                     phone=[] :: [] | binary(),
                     avatar=[] :: [] | binary(),
                     update=[] :: [] | integer(),
                     status=[] :: [] | get | update | set | remove | create | del | add | list | atom()}).
```

```erlang
-record('Contact',  {phone_id=[] :: [] | binary(),
                     avatar=[] :: [] | binary(),
                     names=[] :: [] | binary(),
                     surnames=[] :: [] | binary(),
                     person_id=[] :: [] | binary(),
                     unread=[] :: [] | integer(),
                     update=[] :: [] | integer(),
                     status=[] :: [] | request | authorization | friend | ban | banned | atom()}).
```

Overview
--------

User emits Search request and gets Contacts (which represents the found rosters) packed in Roster
message (the search result envelop).

Protocol
--------

### SEARCH ROSTERS

```
1. client sends `{'Search,RosterId,Phone,_,_,[],contact}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{'Roster,RosterId,_,_,_,UserList,_,_,_,_,_,_}`
             or `{io,{error,profile_not_found},<<>>}`
             or `{io,{error,roster_not_found},<<>>}`
             or `{io,{error,not_authorized},<<>>}`
             to `actions/1/api/:client` once.
```

