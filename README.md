## FEFU MMORPG Protocol — FEMP/0.2

### Abstract

Specification of client-server communication for FEFU MMORPG training project.

### Status of This Memo

This document is a product of collaborative design by students of group B8303A
of Far Eastern Federal University (FEFU) of Russian Federation.
The purpose of this document is to provide students of group B8303A with unified
standard of client-server communication for particular computer science course 
of second semester of 2013-2014 academic year by https://github.com/klenin/

### Table of Contents

- [FEFU MMORPG Protocol — FEMP/0.1](#fefu-mmorpg-protocol-—-femp01)
    - [Abstract](#abstract)
    - [Status of This Memo](#status-of-this-memo)
    - [Table of Contents](#table-of-contents)
    - [Requirements](#requirements)
    - [Introduction](#introduction)
    - [Terminology](#terminology)
    - [Authorization](#authorization)
        - [Register](#register)
            - [Request](#request)
            - [Response](#response)
        - [Login](#login)
            - [Request](#request-1)
            - [Response](#response-1)
        - [Logout](#logout)
            - [Request](#request-2)
            - [Response](#response-2)
    - [Game Interaction](#game-interaction)
        - [Common Invariants](#common-invariants)
        - [Examine](#examine)
            - [Request](#request-3)
            - [Response](#response-3)
        - [Get Dictionary](#get-dictionary)
            - [Request](#request-4)
            - [Response](#response-4)
        - [Logout](#logout-1)
        - [Look](#look)
            - [Request](#request-5)
            - [Response](#response-5)
        - [Move](#move)
            - [Request](#request-6)
            - [Response](#response-6)
        - [Tick](#tick)
    - [Testing](#testing)
        - [Start Testing](#start-testing)
            - [Request](#request-7)
        - [Set Up Constants](#set-up-constants)
            - [Request](#request-8)

### Requirements

The key words `MUST`, `MUST NOT`, `REQUIRED`, `SHALL`, `SHALL NOT`, `SHOULD`,
`SHOULD NOT`, `RECOMMENDED`, `MAY`, and `OPTIONAL` in this document are to be
interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

An implementation is not compliant if it fails to satisfy one or more of the
MUST or REQUIRED level requirements for the protocols it implements. An
implementation that satisfies all the MUST or REQUIRED level and all the SHOULD
level requirements for its protocols is said to be `unconditionally compliant`;
one that satisfies all the MUST level requirements but not all the SHOULD level
requirements for its protocols is said to be `conditionally compliant.`

### Introduction

Client and server communicate with request and response messages.
Each message MUST be represented by a single JSON object.
Messages are sent via either HTTP/1.1 or WebSocket protocol.

Request message MUST contain a single key `action` with a corresponding string
value determining required action.
Each request message MUST be answered with a corresponging response message.

Response message MUST contain a single key `result` with a corresponding value
describing result.

Each response MUST contain key `action` with a value of the same key `action` of
corresponding request. This one serves for describing response type.

If server does not handle the request regardless of underlying transport server
MUST respond with a key `result` of value `badAction`.

Both request and response messages MAY contain any other key/value pairs
specific for particular request/response.

### Terminology

TBD — http://en.wiktionary.org/wiki/TBD

Key in context of json message stands for name in name/value pair which is the
same as key/value pair or attribute/value pair. JSON RFC uses `name` in such
context.

### Authorization

An authorization stage of communication MUST use HTTP/1.1 as an underlying
transport. A method of HTTP request MUST be POST. Message MUST be HTTP
message-body. Content-Type header of HTTP response MUST be `application/json`.

#### Register

The requirements for user credentials are as follows:

login: `[a-zA-Z0-9]{2,36}` i.e. minimal length is 2 symbols and maximum
length is 36 symbols. Allowed charset is latin symbols and numbers.

password: `.{6, 36}` i.e. minimal length is 6 symbols and maximum length is
36 symbols. Any character is allowed except characters indexed from 0 to 31
in ASCII.

##### Request

    action: register
    login: <new client's login> 
    password: <new client's password>

##### Response

    result: [ok, badPassword, badLogin, loginExists]

#### Login

##### Request
 
    action: login
    login: <client's login> 
    password: <client's password>

##### Response

    result: [ok, invalidCredentials]
    sid: <string representation of session identifier>
    webSocket: <WebSocket server URI>
    id: <player ID for use with Game Interaction requests>

#### Logout

##### Request

    action: logout
    sid: <client's sid>

##### Response

    result: [ok, badSid]

### Game Interaction

All communication other than authorization stage (with an exception of logout)
is done via WebSocket protocol as underlying transport.

#### Common Invariants

Those rules apply to any client-server communication of [Game Interaction](#game-interaction)
section. Explicit inclusion of these rules may be ommited. If there is an
exception from the rules, the corresponding section shall state it explicitly.

Each request message sent after client has been logged in MUST have a key `sid`
with a string value of client sid provided by server. In case of invalid sid the
`result` key of response MUST have a value of `badSid`.

In case of successful response `result` key of respone MUST have a value `ok`.

#### Examine

##### Request

    action: examine
    id: <actor's identifier>

##### Response

    result: [ok, badSid, badId]
    id: <actor's id>
    type: <actor's type>
    login: <player's login>
    x: <x coordinate>
    y: <y coordinate>

#### Get Dictionary

Dictionary is a json object describing mapping from game map cell (recieved via
look action) to string value of cell type e.g.

```json
{
    ".": "grass",
    "#": "wall"
}
```

##### Request

    action: getDictionary

##### Response

    dictionary: {...}

#### Logout

See [Logout](#logout)

#### Look

A request for server to provide information for a map area around the client's
player. Such information MUST be provided via keys `map` and `objects`.

TBD: If size of such an area must be standardized

`map` key MUST have a value of an array of array of strings. Such strings are
decoded via dictionary got using `getDictionary` request. `map` 2d array has
row-major order. E.g. for a 4x6 (4 rows, 6 columns) map area:

```json
"map":
[
    ["#", "#", ".", "#", "#", "#"],
    [".", ".", ".", ".", ".", "."],
    [".", ".", ".", ".", ".", "."],
    ["#", "#", "#", ".", "#", "#"]
]
```

`actors` key MUST have a value of an array of objects type. Each element of the
array MUST describe a single actor present at the provided area in the following
form:

    type: <actor's type>
    id: <actor's id>
    x: <global map space x coordinate of actor>
    y: <global map space y coordinate of actor>

##### Request

    action: look

##### Response

    map: [[...]]
    actors: [{...}, ...]
    x: <global map space x coordinate of player's center>
    y: <global map space y coordinate of player's center>

#### Move
    
##### Request

    action: move
    direction: [west, north, east, south]
    tick: <tick number for move action>

##### Response

TBD:

- If client should expect `"result": "ok"` or even any response at all
- If `result` must be distinct from `ok` in case of player trying to move in the
direction of wall right before him

#### Tick

For each simulation tick server MUST broadcast current tick to all the clients.
Tick message is neither request nor response message therefore implicit rules
for keys `sid`, `action`, `result` don't apply to it.

Tick message is a JSON object with a single key `tick` with a value of
broadcasted tick number.

Tick numbers are required to grow monotonously by `1` for each tick.

### Testing

#### Start Testing

This request MUST be sent each time at the beginning of testing stage.

##### Request

    action: startTesting

#### Set Up Constants

Upload a set of constants for a server to immediately set up. There is a
reference set of constants for the purposes of cross server testing:

```json
{
    "action": "setUpConst",
    "playerVelocity": 1.0,
    "slideThreshold": 0.1,
    "ticksPerSecond": 60,
    "screenRowCount": 7,
    "screenColumnCount": 9
}
```

##### Request

```
action: setUpConst
playerVelocity: <a portion of tile player travels through the world per second>
slideThreshold: <a portion of tile which gets ignored when moving towards it>
ticksPerSecond: <a number of simulation cycles per second>
screenRowCount: <a number of tile rows in a rectangle get via `look`>
screenColumnCount: <a number of tile columns in a rectangle get via `look`>
```