ROSTER: Managing Contact and Chat Lists
=======================================

Version 1.0 Maxim Sokhatsky

Endpoints
--------

* `actions/1/api/phone/:phone` — MQTT
* `actions/1/api/:client` — MQTT
* `events/1//api/anon/:client/:token` — MQTT
* `p2p/:phone_id` - MQTT

Tuples
------

The Roster class represents User Impersonified Account or
Contact Book or Contact List of the User. Each account connected to single
Phone or Profile or Device.

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
                     status=[] :: [] | get | update | set | remove |
                                create | del | add | list | last_msg | atom()}).
```

```erlang
-record('Contact',  {phone_id=[] :: [] | binary(),
                     avatar=[] :: [] | binary(),
                     names=[] :: [] | binary(),
                     surnames=[] :: [] | binary(),
                     person_id=[] :: [] | binary(),
                     unread=[] :: [] | integer(),
                     last_msg=[] :: [] | #'Message'{},
                     update=[] :: [] | integer(),
                     presence=[] :: [] | online | offline | atom(),
                     status=[] :: [] | request | authorization | friend
                                      | last_msg | ban | banned | atom()}).
```

* person_id — VoxImplant User Id

Overview
--------

ROSTER API manages different accounts with different contact lists.

Protocol
--------

### `Roster/get` — Get Roster

```
1. client sends `{'Roster,Id,_,_,_,_,_,_,_,_,_,get}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{'Roster,Id,_,_,_,_,_,_,_,_,_,_}`
             or `{io,{error,roster_not_found},<<>>}`
             or `{io,{error,not_authorized},<<>>}`
             to `actions/1/api/:client` once.
```

### `Roster/update` — Update Roster

```
1. client sends `{'Roster,Id,_,_,_,[],[],_,_,_,UpdateTime,update}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{'Roster,Id,_,_,_,UpdateContacts,UpdateRooms,_,_,_,NewUpdateTime,update}`
             or `{io,{error,roster_not_found},<<>>}`
             or `{io,{error,not_authorized},<<>>}`
             to `actions/1/api/:client` once.
```

### `Roster/set` or `Roster/patch` — Set Roster

```
1. client sends `{'Roster,Id,_,_,_,_,_,_,_,_,_,set}`
             to `events/1/:node/api/anon/:client/:token` once.
```

```
2. server sends `{'Roster,Id,_,_,_,_,_,_,Phone,_,NewUpdateTime,set}`
             or `{io,{error,not_authorized},<<>>}`
             to `actions/1/api/phone/:phone` once.
```

```
3. server sends `{Contact,Phone+Id,_,_,_,_,_,_,NewUpdateTime,_}`
             to `p2p/:phone_id` once.
```

### `Roster/remove` — Remove Roster

```
1. client sends `{'Roster,Id,_,_,_,_,_,_,_,_,_,remove}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{io,{error,not_authorized},<<>>}`
             or `{io,{error,profile_not_found},<<>>}`
             or `{io,{error,roster_not_found},<<>>}`
             or `{io,{ok,removed},<<>>}`
             to `actions/1/api/:client` once.
```

```
3. server sends `{Contact,Phone+Id,_,_,_,_,_,NewUpdateTime,remove}`
                to `p2p/:phone_id` once.
```

### `Roster/create` — Create Roster

```
1. client sends `{'Roster,Id,_,_,_,_,_,_,_,create}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{'Roster,Id,_,_,_,_,_,_,_,_,_,_}`
             or `{io,{error,not_authorized},<<>>}`
             to `actions/1/api/:client` once.
```

### `Roster/list` — List Rosters

```
1. client sends `{'Roster,_,_,_,_,_,_,_,_,Phone,_,list}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{io,List,<<>>}`
             or `{io,{error,not_authorized},<<>>}`
             to `actions/1/api/:client` once.
```

### `Roster/add` — Add Roster Contacts

```
1. client sends `{'Roster,Id,_,_,_,List,_,_,_,_,_,add}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{io,{error,roster_not_found},<<>>}`
             or `{io,{error,not_authorized},<<>>}`
             or `{io,{ok,added},<<>>}`
             or `{io,{ok2, already_present,_},<<>>}`
             to `actions/1/api/:client` once.
```

### `Roster/del` — Delete Roster Contacts

```
1. client sends `{'Roster,Id,_,_,_,List,_,_,_,_,_,del}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{io,{error,roster_not_found},<<>>}`
             or `{io,{error,not_authorized},<<>>}`
             or `{io,{error,contacts_not_found},<<>>}`
             or `{io,{ok2,removed,List},<<>>}`
             to `actions/1/api/:client` once.
```
