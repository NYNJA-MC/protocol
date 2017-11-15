AUTH: User Registration
=======================

Version 2.0 Maxim Sokhatsky, Liubov Mykhailova, Yuri Maslovsky

Endpoints
--------

* `actions/1/api/:client` — MQTT
* `events/1//api/anon//` — MQTT

Tuples
------

Auth tuple represents token storage instances.

```erlang
-record('Auth',     {client_id =[] :: [] | binary(),
                     dev_key   =[] :: [] | binary(),
                     user_id   =[] :: [] | binary(),
                     phone     =[] :: [] | binary(),
                     token     =[] :: [] | binary(),
                     type      =[] :: [] | atom(),
                     sms_code  =[] :: [] | binary(),
                     attempts  =[] :: [] | integer(),
                     services  =[] :: list(atom()),
                     push      =[] :: [] | binary(),
                     os        =[] :: [] | atom() | ios | android | web,
                     created   =[] :: [] | integer() | binary(),
                     last_online=[]:: [] | integer() }).
```

AUTH model represents distinct non-duplicating 10-tuples with
single unique key=token which represent session identifier.

Client Id
---------

`<<"emqttd_",Symbols/binary>>` or `<<"reg_",Symbols/binary>>` if registration

ClientId resembles all tuple fields to represent a unique
device identifier connected with an given token.
ClientId is an unique MQTT session identifier.
ClientId must be `<<"emqttd_reg_",Symbols/binary>>` and respawned with
clean_session to `true` if registration process starts. After
verify the client gets the new ClientId without `_reg_` suffix
(`<<"emqttd_",Symbols/binary>>`), Token and must disconnect and connect
with the new ClientId and clean_session to `false`. The mqtt client
password is the received Token. If connect is ok then the user is login.


```
TOKEN PHONE      DEVKEY CLIENTID OS      SERVICES
12131 3800001122 APPLE1 emqttd_1 iOS     [ua]
10293 3800000000 NEXUS1 emqttd_2 Android [jwt]
10294 3800000001 NEXUS2 emqttd_3 Android [jwt]
12323 3800000000 SONY02 emqttd_4 Android [jwt]
```

Overview
--------

AUTH API is dedicated for process of user registration via external confirmation providers.

Protocol
--------

### `Auth/reg` — Registration

New clients should send `Auth/reg` registration request before start using the system.
Server will store the `token` and `client_id` fields of `Auth` record and database
and issue a SMS or JWT authentication mechanism. To verify device user then should send
`Auth/voice` for IVR verification or `Auth/verify` for SMS verification.

```
1. client sends `{'Auth',<<"reg_", _/binary>> =ClientId,[],[],Phone,[],reg,[],[],Services,[],[],[],[]}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{io, Result, <<>>}`
             or `{io, {ok2, jwt_sent, JwtCode}, <<>>}`
             to `actions/1/api/:client` once.
```

Result:

* `Services` - `[]` or `[jwt]`.
* `{ok,sms_sent}` or `{ok2, jwt_sent, JwtCode}` — the genrated SMS code or JWT code is sent successfully.

### `Auth/voice` — Voice Call

The are several channels of verification.
`Auth/voice` API is dedicated for IVR confirmation.

```
1. client sends `{'Auth',ClientId,[],[],[],[],voice,[],[],[Lang],[],[],[],[]}`
             to `events/1//api/anon//` once.
```

Lang:

* ua, en, it — ANSI country codes as atoms

```
2. server sends `{io, Result, {'Auth', Token}}`
             to `actions/1/api/:client` once.
```

Result:

* `{ok,call_in_progress}` — Call started
* `{error,session_not_found}` — Auth record not found

### `Auth/resend` — Resend

In case of error client might want to send resend SMS for alredy registered token.

```
1. client sends `{'Auth',<<"reg_", _/binary>> =ClientId,[],[],[],[],resend,[],[],[],[],[],[],[]}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{io, Result, <<>>}`
             to `actions/1/api/:client` once.
```

Result:

* `{ok,sms_sent}` — the genrated SMS code is sent successfully.
* `{error,session_not_found}` — Auth record absent.
* `{error,mismatch_user_data}` — Auth record found but wrong.

### `Auth/verify` — Verify

Verify that SMS you've entered and the one we sent you are same.

```
1. client sends `{'Auth',<<"reg_", _/binary>> =ClientId,[],[],[],[],verify,SMS,[],Service,[],[],[],[]}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{io, Result, <<>>}`
             to `actions/1/api/:client` once and
   server sends `{'Profile', Phone, Services, Rosters,_,_,_,_,_}`
             to `actions/1/api/:client` once.

```

Result:

* `{ok2,login, {'Client', ClientId}}` — Logged in
* `{error,session_not_found}` — Auth record absent
* `{error,mismatch_user_data}` — Record is found but wrong
* `{error,invalid_sms_code}` — Wrong SMS
* `{error,invalid_jwt_code}` — Wrong SMS if `jwt` in services
* `{error,attempts_expired}` — Hacker?

### `Auth/login` — Login

Proceed Authentication with a given credentials.
Client sends Token as MQTT password,
`ClientId = <<"emqttd_", _/binary>>`
```
1. server sends `{io, Result, <<>>}`
             to `actions/1/api/:client` once.
             or nothing if token is not expired
```

Result:

* `{ok2,login, {ClientId, Token}}` — Logged in if token has been expired

### `Auth/logout` — Logout

Logging out means your device token removal. You'll need to be reregistered on this device.

```
1. client sends `{'Auth',[],[],[],[],[],logout,[],[],[],[],[],[],[]}`
             to `events/1//api/anon//` once.
```

```
2. server sends `{io, Result, {'Auth', Token}}`
             to `actions/1/api/:client` once.
```

Result:

* `{ok,logout}` — Logged out

### `Auth/push` — Write Google or Apple token to Auth table

```
1. client sends `{'Auth',[],[],[],[],[],push,[],Push,OS,[],[]}`
             to `events/1//api/anon/:client/:token` once.
```

* OS — android, ios, web
* Push — OS specific Push token

```
2. server sends `{io, Result, <<>>}`
             to `actions/1/api/:client` once.
```

Result:

* `<<>>` — OK
* `{error,mismatch_user_data}` — Record is found but wrong

### `Auth/list` — List of client sessions

```
1. client sends `{'Auth',[],[],[],[],[],list,[],[],[],[],[]}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{'Auth',_,_,_,_,_,_,_,_,_,_,_}`
             to `actions/1/api/:client` number of sessions times.
```

### Invalid Messages

```
1. client sends `{'Auth',_,_,_,_,_,_,_,_,_,_,_}`
             to `events/1//api/anon/:client/:token` once.
```

```
2. server sends `{io, {error,invalid_data}, <<>>}`
             to `actions/1/api/:client` once.
```

