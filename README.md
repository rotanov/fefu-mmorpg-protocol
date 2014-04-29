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

- [FEFU MMORPG Protocol — FEMP/0.2](#fefu-mmorpg-protocol-—-femp02)
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
        - [Attack](#attack)
            - [Request](#request-7)
        - [Tick](#tick)
            - [Possible Events](#possible-events)
                - [Attack](#attack-1)
    - [Testing](#testing)
        - [Start Testing](#start-testing)
            - [Request](#request-8)
            - [Response](#response-7)
        - [Stop Testing](#stop-testing)
            - [Request](#request-9)
            - [Response](#response-8)
        - [Set Up Constants](#set-up-constants)
            - [Request](#request-10)
            - [Response](#response-9)
        - [Get Constants](#get-constants)
            - [Request](#request-11)
            - [Response](#response-10)
        - [Set Up Map](#set-up-map)
            - [Request](#request-12)
            - [Response](#response-11)

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

Request message MUST contain a key `action` with a corresponding string value
determining required action.

Each request message MUST be answered with a corresponding response message.

Response message MUST contain a key `result` with a corresponding value
describing result.

Each response MUST contain key `action` with a value of the same key `action` of
corresponding request. This one serves for describing response type.

If server does not handle the request regardless of underlying transport and
with regard to value of 'action' key then server MUST respond with a key
`result` of value `badAction`.

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
    type: <actor's type. It may be one of `player`, `monster`, `item` for now>
    login: <player's login. MAY be present if `type` is `player`>
    x: <x coordinate>
    y: <y coordinate>

If type is either `player` or `monster` response MAY contain the following:

    health: <actor's current number of health points>
    maxHealth: <actor's maximum number of health points>

If `type` is `monster` response MAY contain these fields:
    
    name: <name of a monster>
    mobType: <string describing the type of a monster>

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

TBD: If size of such area must be standardized

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

    type: <actor's type. May be one of `player`, `monster`, `item`>
    id: <actor's id>
    x: <global map space x coordinate of actor>
    y: <global map space y coordinate of actor>

If `type` is not `item` actor description MUST contain:

    health: <actor's current number of health points>
    maxHealth: <actor's maximum number of health points>

If `type` is `monster` actor description MUST contain these fields:

    mobType: <string describing the type of a monster>
    
TBD: list of possible mobType values

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

#### Attack

Request of a clent performing an attack at a specified coordinates in a global
map space.

##### Request

    action: attack
    target: <an array of two elements - x, y coordinates of an attack>

#### Tick

For each simulation tick server MUST broadcast current tick to all the clients.
Tick message is neither request nor response message therefore implicit rules
for keys `sid`, `action`, `result` don't apply to it.

Tick message is a JSON object. It MUST contain key `tick` with a value of server
current tick number.

Tick numbers are required to grow monotonously by `1` for each tick.

Tick message MAY also contain an array `events` of JSON objects describing
events occured at a given tick. 

Server MAY decide to send some events only for a subset of clients. e.g. attack
action only to ones for which it is visible.

Presently there is only one event `attack`.

##### Possible Events

###### Attack

TBD: what a `blowType` actually means
TBD: should we really send target id or it may be reasonable to sent [x, y] of
an attack
TBD: what should be sent if there are multiple targets for a single attack

Field `killed` may be omitted if it is `false`.

    event: "attack"
    attacker: <attacker's id>
    target: <target's id>
    blowType: <string describing attack type e.g. "CLAW", "BITE", etc>
    dealtDamage: <damage dealt>
    killed: <whether target was killed or not>

### Testing

There are a number of request messages available only when server is in the
testing stage. Such messages are marked with "Testing stage only." If such 
message to be sent while testing stage is not active, server MUST respond with
`"result": "badAction"`.

#### Start Testing

This request MUST be sent each time at the beginning of testing stage.
Once this message is responded with `"result": "ok"`, it is valid to state
that testing stage is now active.

It is invalid to request Start Testing when testing stage is already active. In
such case request MUST be answered with `"result": "badAction"`.

##### Request

    action: startTesting

##### Response

    result: [ok, badAction]

#### Stop Testing

Testing stage only.

Each testing stage MUST be closed with this request. Once responded with
`"result": "ok"` it is valid to state that server is no more in the testing
stage.

##### Request

    action: stopTesting

##### Response

    result: [ok, badAction]

#### Set Up Constants

Testing stage only.

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

##### Response

    result: [ok, badAction]

#### Get Constants

Get a set of current constant values.

##### Request

    action: getConst

##### Response

```
result: ok
playerVelocity: <value>
slideThreshold: <value>
ticksPerSecond: <value>
screenRowCount: <value>
screenColumnCount: <value>
```

#### Set Up Map

Testing stage only.

Upload map to server and set it up as currently active. If server assumes
uploaded map to be invalid (e.g. malformed map array or wrong cell value) then
the request MUST be responded with `badMap`. Innermost values of `map` key are
those found in [Dictionary](#get-dictionary).

##### Request

```json
{
    "action": "setUpMap",
    "map": [
    [<column count number of values>],
    [<column count number of values>],
    [<column count number of values>],
    ... <total of row count arrays with row values>
    [<column count number of values>]
    ]
}
```

##### Response

    result: [ok, badMap, badAction]