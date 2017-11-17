FRIEND: Manage Private Subscriptions
====================================

Version 1.0 Maxim Sokhatsky

Endpoints
---------

* `actions/1/api/:client` — MQTT
* `events/1//api/anon//`  — MQTT
* `ses/:phone`            — MQTT

Tuples
------

```erlang
-record('Friend', {phone_id = [] :: [] | binary(),
                  friend_id = [] :: [] | binary(),
		          status    = [] :: [] | contact | simple
                                       | list | common_groups | ban | unban
                                       | mute | unmute
                                       | request | confirm | revoke }).
```

Overview
--------

FRIEND API serves the management of peer-to-peer private subscriptions.
At the time users establish a friendship the MQTT subscriptions are being created.

Protocol
--------

### `Friend/request` — Friendship Request

```
1. client sends `{'Friend',PhoneId,FriendId,request}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{'Contact',_,_,_,_,_,_,_,_,_,_,_,_,_,request}`
             to `ses/:party` once.
```

```
3. server sends `{'Contact',_,_,_,_,_,_,_,_,_,_,_,_,_,autorization}`
             to `ses/:counterparty` once.
```

### `Friend/confirm` — Confirm friendship

This is the moment when subscriptions to private
conversation MQTT topic are being created for both counterparties.

```
1. client sends `{'Friend',PhoneId,FriendId,confirm}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{io,{ok,added},{ok,added}}`
             or `{io,{ok,added},{error,contact_not_found}}`
             or `{io,{error,contact_not_found},{ok,added}}`
             or `{io,{error,contacts_not_found},{error,contacts_not_found}}`
             to `actions/1/api/:client` once.
```

```
3. server sends `{'Contact',_,_,_,_,_,_,_,_,_,_,_,_,_,friend}`
             to `ses/:party` once
            and `ses/:counterparty` once.
```

### `Friend/revoke` — Revoke friendship

This is the moment when subscriptions to private
conversation MQTT topic are being removed for both counterparties.

```
1. client sends `{'Friend',PhoneId,FriendId,revoke}`
             to `events/1//api/anon//` once.
```

```
3. server sends `{'Contact',_,_,_,_,_,_,_,_,_,_,_,_,_,revoke}`
             to `ses/:party` once.
            and `ses/:counterparty` once.
```

### `Friend/ban` — Ban friend

```
1. client sends `{'Friend',PhoneId,FriendId,ban}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{io,{ok,added},{ok,added}}`
	         or `{io,{ok,added},{error,contact_not_found}}`
             or `{io,{error,contact_not_found},{ok,added}}`
             or `{io,{error,contacts_not_found},{error,contacts_not_found}}`
             to `actions/1/api/:client` once.
```

```
3. server sends `{'Contact',_,_,_,_,_,_,_,_,_,_,_,_,_,banned}`
             to `ses/:party` once.
            and `ses/:counterparty` once.
```

### `Friend/unban` — Unban friend

```
1. client sends `{'Friend',PhoneId,FriendId,unban}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{'Contact',_,_,_,_,_,_,_,_,_,_,_,_,_,unbanned}`
             to `ses/:party` once.
            and `ses/:counterparty` once.
```

### `Friend/mute` — Mute friend

```
1. client sends `{'Friend',PhoneId,FriendId,mute}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{'Contact',_,_,_,_,_,_,_,_,_,_,_,_,_,muted}`
             to `ses/:party` once.
```

### `Friend/unmute` — Unute friend

```
1. client sends `{'Friend',PhoneId,FriendId,unmute}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{'Contact',_,_,_,_,_,_,_,_,_,_,_,_,_,unmuted}`
             to `ses/:party` once.
```
