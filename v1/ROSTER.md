ROSTER: Managing Contact and Chat Lists
=======================================

Version 1.0 Maxim Sokhatsky

Endpoints
--------

* `actions/1/api/:client` — MQTT
* `events/1//api/anon//`  — MQTT
* `ses/:phone`            — MQTT
* `ac/:phone_id`          — MQTT
* `muc/:phone_id`         — MQTT

Tuples
------

The Roster class represents User Impersonified Account or
Contact Book or Contact List of the User. Each account connected to single
Phone or Profile or Device.

```erlang
-record('Roster',   {id      =[] :: [] | integer(),
                     names   =[] :: [] | binary(),
                     surnames=[] :: [] | binary(),
                     email   =[] :: [] | binary(),
                     nick    =[] :: [] | binary(),
                     userlist=[] :: list(#'Contact'{}),
                     roomlist=[] :: list(#'Room'{}),
                     favorite=[] :: list(#'Star'{}),
                     tags    =[] :: list(#'Tag'{}),
                     phone   =[] :: [] | binary(),
                     avatar  =[] :: [] | binary(),
                     update  =0  :: [] | integer(),
                     status  =[] :: [] | get | set | create | del | remove
                                       | add | update | list | patch | last_msg | atom()}).
```

```erlang
-record('Contact',  {phone_id  =[] :: [] | binary(),
                     avatar    =[] :: [] | binary(),
                     names     =[] :: [] | binary(),
                     surnames  =[] :: [] | binary(),
                     nick      =[] :: [] | binary(),
                     email     =[] :: [] | binary(),
                     vox_id    =[] :: [] | binary(),
                     reader    = 0 :: [] | integer(),
                     unread    = 0 :: [] | integer(),
                     last_msg  =[] :: [] | #'Message'{},
                     update    = 0 :: [] | integer(),
                     settings  =[] :: [] | list(#'Feature'{}),
                     presence  =[] :: [] | atom(),
                     status    =[] :: [] | request | authorization | internal
                                         | friend | last_msg | ban | banned | deleted }).

```

* person_id — VoxImplant User Id

Overview
--------

ROSTER API manages different accounts with different contact lists.

Protocol
--------

### `Roster/get` — Get Roster

```
1. client sends `{'Roster,Id,_,_,_,_,_,_,_,_,_,_,_,get}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{'Roster,Id,_,_,_,_,_,_,_,_,_,_,_,_}`
             or `{io,{error,roster_not_found},<<>>}`            
             to `actions/1/api/:client` once.
```

### `Roster/patch` — Update Roster

```
1. client sends `{'Roster,Id,_,_,Email,Nick,Contacts,Rooms,Stars,Tags,Phone,Avatar,Time,patch}`
             to `events/1//api/anon//` once.
```

```

2. server sends `{'Roster,Id,_,_,_,_,_,_,_,_,_,_,_,_}`
             to `ses/:phone` once
             or `{io,{error,not_found},<<>>}`
             to `actions/1/api/:client` once.
```

```
3. server sends `{Contact,PhoneId,_,_,_,_,_,_,_,_,_,Time,_,_,_}`
             to `ac/:phone_id` once
             to `muc/:phone_id` once.

```

### `Roster/nick` — Update Nick

```
1. client sends `{'Roster,Id,_,_,_,Nick,_,_,_,_,_,_,_,nick}`
             to `events/1//api/anon//` once.
```

```

2. server sends `{'Roster,Id,_,_,_,_,_,_,_,_,_,_,_,_}`
             to `ses/:phone` once
             or `{io,{error,not_found},<<>>}`
             or `{io,{error,nick},<<>>}`            - nick is already in use by another user
             or `{io,{error,invalid_nick},<<>>}`    - wrong nick format
             to `actions/1/api/:client` once.

```

```
3. server sends `{Contact,PhoneId,_,_,_,_,_,_,_,_,Time,_,_,_}`
             to `ac/:phone_id` once.
```

### `Roster/remove` — Remove Roster

```
1. client sends `{'Roster,Id,_,_,_,_,_,_,_,_,_,_,_,remove}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{io,{error,profile_not_found},<<>>}`
             or `{io,{error,roster_not_found},<<>>}`
             or `{io,{ok,removed},<<>>}`
             to `actions/1/api/:client` once.
```

```
3. server sends `{Contact,PhoneId,_,_,_,_,_,_,_,_,Time,_,_,remove}`
             to `ac/:phone_id` once
             to `muc/:phone_id` once.  .
```

### `Roster/create` — Create Roster

```
1. client sends `{'Roster,Id,_,_,_,_,_,_,_,_,_,_,_,create}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{'Roster,Id,_,_,_,_,_,_,_,_,_,_,_,_}`
             to `actions/1/api/:client` once.
```

### `Roster/list` — List Rosters

```
1. client sends `{'Roster,_,_,_,_,_,_,_,_,_,_,_,_,list}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{io,List,<<>>}`
             or `{io,{error,profile_not_found},<<>>}`
             to `actions/1/api/:client` once.
```

### `Roster/add` — Add Roster Contacts

```
1. client sends `{'Roster,Id,_,_,_,_,List,_,_,_,_,_,_,add}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{io,{error,roster_not_found},<<>>}`
             or `{io,{ok,added},<<>>}`
             or `{io,{ok2, already_present,_},<<>>}`
             to `actions/1/api/:client` once.
```

### `Roster/del` — Delete Roster Contacts

```
1. client sends `{'Roster,Id,_,_,_,_,List,_,_,_,_,_,_,del}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{io,{error,roster_not_found},<<>>}`
             or `{io,{error,contacts_not_found},<<>>}`
             or `{io,{ok2,removed,List},<<>>}`
             to `actions/1/api/:client` once.
```
