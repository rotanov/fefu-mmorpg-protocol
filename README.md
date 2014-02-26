fefu-mmorpg-protocol
====================

A protocol specification for fefu-mmorpg project

Client sends the body to the server with a single json-object by HTTP 1.1
protocol. This object containts keys (action,login,password,sid), which define
requested action.

Answer from server mime-type/json is a json-object, which contains keys
(result, sid, websocket). Value of key result is a string "ok", otherwise it
will contatin a error message.

Register
========
Request

action: register
login: <new user's login> 
password: <new user's password>

Response

result: [ok,bad_password,bad_login,login_exists]
sid: <random generated sid as string>
websocket: <>

Login
=====

Request
 
action: login
login: <user's login> 
password: <user's password>

Response

result: [ok,wrong_password]
sid: <random generated sid as string>

Logout
======

Request

action: logout
sid: <user's sid>

Response

result: [ok,bad_sid]



