AUTH: Contact Registration
==========================

Version 1.0 Maxim Sokhatsky

Endpoints
--------

* `actions/1/api/:client` — MQTT
* `events/1//api/anon/:client/:token` — MQTT

Tuples
------

Auth tuple represents token storage instances.

```erlang
-record('Auth',     {token=[] :: [] | binary(),
                     dev_key=[] :: [] | binary(),
                     user_id=[] :: [] | binary(),
                     phone=[] :: [] | binary(),
                     client_id=[] :: [] | binary(),
                     type=[] :: [] | atom(),
                     sms_code=[] :: [] | binary(),
                     attempts=[] :: [] | integer(),
                     services=[] :: list(atom()),
                     created = [] :: [] | integer() | binary(),
                     last_online = [] :: [] | integer() | binary()}).
```

AUTH model represents distinct non-duplicating 10-tuples with
single unique key=token which represent session identifier.

Client Id
---------

* ClientId = `"emqttd_3800000001_NEXUS1_Android_ua"`

ClientId resembles all tuple fields to represent a unique
row indexed naturally by token. ClientId is an unique MQTT
session identifier that respawned with clear_session to `false`.
Also ClientId should resemble all session columns:

```
UNIQUE TOKEN PHONE      DEVKEY CLIENT  SERVICES
       12131 3800001122 APPLE1 iOS     [ua]
       10293 3800000000 NEXUS1 Android [ua,jwt]
       10294 3800000001 NEXUS1 Android [ua,jwt]
       12323 3800000000 SONY02 Android [en,jwt]
```

Overview
--------

AUTH API is dedicated for process of user registration with SMS confirmation.

Protocol
--------

### Auth Issuing / SMS sending

```
1. client sends `{'Auth',Token,_,ClientId,Phone,_,Type,_,_,Services,_,_}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{io, ResultType, {Table, Token}}`
             to `actions/1/api/:client` once.
```

Table: atom constant `Auth` <br>
Token: binary string like: `<<"1fcac6caea64dd55d2cba",_/binary>>` <br>
ResultType: <br>

* `{ok,login}` — the session is verified.
* `{ok,sms_send}` — the verified sms is sent successfully.
* `{ok2,jwt,Code}` — the verified jwt code is generated if was requested with `jwt` in Services.
* `{error,sms_send}` — verified sms is not sent successfully.
* `{error,not_verified}` —  the session is not verified.
* `{error,mismatch_user_data}` — the devkey and the phone are not matched for the session.
* `{error,session_not_found}` — the session with the token is not found.
* `{error,miltiple_devices}` — more then one sessions with the same device key is found.


### User Confirmation

```
1. client sends `{'Auth',Token,_,ClientId,Phone,_,verify,SMS,_,_,_,_}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{io, ResultType, {Table, Token}}`
             to `actions/1/api/:client` once.
```

Table: atom constant `Auth` <br>
Token: binary string like: `<<"1fcac6caea64dd55d2cba",_/binary>>` <br>
ResultType: <br>

* `{error,attempts_expired}` — the number of attempts is expired
* `{error,roster_not_found}` — nothing rosters found in the user profile
* `{ok,login}` — already on the state on sending sms
* `{error,session_not_found}` — Auth record not found
* `{error,invalid_sms_code}` — mistyping SMS

### Resend User Confirmation

```
1. client sends `{'Auth',Token,_,ClientId,Phone,_,resend_sms,_,_,_,_,_}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{io, ResultType, {Table, Token}}`
             to `actions/1/api/:client` once.
```

Table: atom constant `Auth` <br>
Token: binary string like: `<<"1fcac6caea64dd55d2cba",_/binary>>` <br>
ResultType: <br>

* `{error,roster_not_found}` — nothing rosters found in the user profile
* `{error,session_not_found}` — Auth record not found
* `{error,sms_send}` — the verified sms is not sent.
* `{ok,login}` — already on the state on sending sms
* `{ok,sms_send}` — the verified sms is sent successfully.


### Request Voice Call

```
1. client sends `{'Auth',Token,_,_,_,_,voice_call,_,_,Services,_,_}`
             to `events/1//api/anon/:client/:token` once.
```

Services: should include language atom. E.g: `[ua,jwt]` or `[en]`.

```
2. server sends `{io, ResultType, {Table, Token}}`
             to `actions/1/api/:client` once.
```

Table: atom constant `Auth` <br>
Token: binary string like: `<<"1fcac6caea64dd55d2cba",_/binary>>` <br>
ResultType: <br>

* `{ok,login}` — already logged in
* `{ok,call_in_progress}` — call in the air
* `{error,session_not_found}` — Auth record is not found
