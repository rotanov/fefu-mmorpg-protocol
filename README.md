# FEFU MMORPG Protocol — FEMP/0.3

# Abstract

Specification of client-server communication for FEFU MMORPG training project.

# Status of This Memo

This document is a product of collaborative design by students of group B8303A
of Far Eastern Federal University (FEFU) of Russian Federation.
The purpose of this document is to provide students of group B8303A with unified
standard of client-server communication for particular computer science course 
of second semester of 2013-2014 academic year by https://github.com/klenin/

# Table of Contents

- [FEFU MMORPG Protocol — FEMP/0.3](#fefu-mmorpg-protocol-—-femp03)
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
  - [Attack](#attack)
    - [Request](#request-3)
  - [Destroy Item](#destroy-item)
    - [Request](#request-4)
  - [Drop](#drop)
    - [Request](#request-5)
    - [Response](#response-3)
  - [Equip](#equip)
    - [Request](#request-6)
    - [Response](#response-4)
  - [Examine](#examine)
    - [Request](#request-7)
    - [Response](#response-5)
  - [Get Dictionary](#get-dictionary)
    - [Request](#request-8)
    - [Response](#response-6)
  - [Logout](#logout-1)
  - [Look](#look)
    - [Request](#request-9)
    - [Response](#response-7)
  - [Move](#move)
    - [Request](#request-10)
  - [Pick Up](#pick-up)
    - [Request](#request-11)
  - [Tick](#tick)
    - [Possible Events](#possible-events)
      - [Attack](#attack-1)
      - [Effect](#effect)
  - [Unequip](#unequip)
    - [Request](#request-12)
  - [Use](#use)
    - [Request](#request-13)
    - [Response](#response-8)
- [Testing](#testing)
  - [Start Testing](#start-testing)
    - [Request](#request-14)
    - [Response](#response-9)
  - [Stop Testing](#stop-testing)
    - [Request](#request-15)
    - [Response](#response-10)
  - [Set Up Constants](#set-up-constants)
    - [Request](#request-16)
    - [Response](#response-11)
  - [Get Constants](#get-constants)
    - [Request](#request-17)
    - [Response](#response-12)
  - [Set Up Map](#set-up-map)
    - [Request](#request-18)
    - [Response](#response-13)
- [Data Invariants](#data-invariants)
  - [Game Objects](#game-objects)
    - [Player](#player)
      - [Slots](#slots)
    - [Monster](#monster)
    - [Item](#item)
    - [Projectile](#projectile)

# Requirements

The key words `MUST`, `MUST NOT`, `REQUIRED`, `SHALL`, `SHALL NOT`, `SHOULD`,
`SHOULD NOT`, `RECOMMENDED`, `MAY`, and `OPTIONAL` in this document are to be
interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

An implementation is not compliant if it fails to satisfy one or more of the
MUST or REQUIRED level requirements for the protocols it implements. An
implementation that satisfies all the MUST or REQUIRED level and all the SHOULD
level requirements for its protocols is said to be `unconditionally compliant`;
one that satisfies all the MUST level requirements but not all the SHOULD level
requirements for its protocols is said to be `conditionally compliant.`

# Introduction

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

# Terminology

TBD — http://en.wiktionary.org/wiki/TBD

Key in context of json message stands for name in name/value pair which is the
same as key/value pair or attribute/value pair. JSON RFC uses `name` in such
context.

# Authorization

An authorization stage of communication MUST use HTTP/1.1 as an underlying
transport. A method of HTTP request MUST be POST. Message MUST be HTTP
message-body. Content-Type header of HTTP response MUST be `application/json`.

## Register

The requirements for user credentials are as follows:

login: `[a-zA-Z0-9]{2,36}` i.e. minimal length is 2 symbols and maximum
length is 36 symbols. Allowed charset is latin symbols and numbers.

password: `.{6, 36}` i.e. minimal length is 6 symbols and maximum length is
36 symbols. Any character is allowed except characters indexed from 0 to 31
in ASCII.

### Request

    action: register
    login: <new client's login> 
    password: <new client's password>

### Response

    result: [ok, badPassword, badLogin, loginExists]

## Login

### Request
 
    action: login
    login: <client's login> 
    password: <client's password>

### Response

    result: [ok, invalidCredentials]
    sid: <string representation of session identifier>
    webSocket: <WebSocket server URI>
    id: <player ID for use with Game Interaction requests>

## Logout

### Request

    action: logout
    sid: <client's sid>

### Response

    result: [ok, badSid]

# Game Interaction

All communication other than authorization stage (with an exception of logout)
is done via WebSocket protocol as underlying transport.

## Common Invariants

Those rules apply to any client-server communication of [Game Interaction](#game-interaction)
section. Explicit inclusion of these rules may be ommited. If there is an
exception from the rules, the corresponding section shall state it explicitly.

Each request message sent after client has been logged in MUST have a key `sid`
with a string value of client sid provided by server. In case of invalid sid the
`result` key of response MUST have a value of `badSid`.

In case of successful response `result` key of respone MUST have a value `ok`.

## Attack

Request of a clent performing an attack at a specified coordinates in a global
map space.

### Request

    action: attack
    target: <an array of two elements - x, y coordinates of an attack>

## Destroy Item

Destroys an item. It is possible if distance between item and player centers
doesn't exceed pickUpRadius or item is already in player's inventory. Otherwise
result MUST be `badId`. If object with this id is not an item result MUST be
`badId`.

### Request

    action: destroyItem
    id: <item's id>

## Drop

Drop item from inventory to the ground.

### Request

    action: drop
    id: <id>

### Response

    result: [ok, badSid, badId]

## Equip

Equips item of given id onto specified slot. If it is impossible to equip this
item onto that slot badSlot is returned. If slot is already occupied then item
which is already there gets automatically unequipped and takes of the item being
equipped.

### Request

    action: equip
    id: <item's id>
    slot: <slot to equip onto>

### Response

    result: [ok, badId, badSid, badSlot]

## Examine

### Request

    action: examine
    id: <actor's identifier>

### Response

    result: [ok, badSid, badId]
    id: <actor's id>
    type: <actor's type. One of `player`, `monster`, `item`, `projectile`>
    login: <player's login. MAY be present if `type` is `player`>
    x: <x coordinate>
    y: <y coordinate>

If type is either `player` or `monster` response MAY contain the following:

    health: <actor's current number of health points>
    maxHealth: <actor's maximum number of health points>

If `type` is `monster` response MAY contain these fields:
    
    name: <name of a monster>
    mobType: <string describing the type of a monster>

## Get Dictionary

Dictionary is a json object describing mapping from game map cell (recieved via
look action) to string value of cell type e.g.

```json
"dictionary":
{
    ".": "grass",
    "#": "wall"
}
```

### Request

    action: getDictionary

### Response

    dictionary: {...}

## Logout

See [Logout](#logout)

## Look

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

    type: <actor's type. One of `player`, `monster`, `item`, `projectile`>
    id: <actor's id>
    x: <global map space x coordinate of actor>
    y: <global map space y coordinate of actor>

If `type` is not `item` actor description MUST contain:

    health: <actor's current number of health points>
    maxHealth: <actor's maximum number of health points>

If `type` is `monster` actor description MUST contain these fields:

    mobType: <string describing the type of a monster>
    
TBD: list of possible mobType values

### Request

    action: look

### Response

    map: [[...]]
    actors: [{...}, ...]
    x: <global map space x coordinate of player's center>
    y: <global map space y coordinate of player's center>

## Move

If total weight of items in Player's inventory exceeds this Player's carrying
capacity then result is `tooHeavy`.
    
### Request

    action: move
    direction: [west, north, east, south]
    tick: <tick number for move action>

## Pick Up

Request for client player to pick up an item with given ID. Object with this ID
MUST have type `item` and distance between player's center and item's center
MUST be less or equal than constant pickUpRadius. Otherwise nothing is picked up
and result is `badId`.

If total weight of Player's inventory exceeds Player's carrying capacity after
picking up an item and it is possible to pick up item then result MUST be
`tooHeavy`.

### Request 

    action: pickUp
    id: <item-to-pick-up's id>

## Tick

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

### Possible Events

#### Attack

If there are multiple targets for a single attack then server MUST produce
multiple `attack` events for each target.

Field `killed` may be omitted if it is `false`.

    event: "attack"
    attacker: <attacker's id>
    target: <target's id>
    blowType: <string describing attack type e.g. "CLAW", "BITE", etc>
    dealtDamage: <damage dealt>
    killed: <whether target was killed or not>

#### Effect

    x: <global-space x coordinate of effect center>
    y: <global-space y coordinate of effect center>
    radius: <visual effect radius>
    type: <type of effect>

## Unequip

Unequips item with given id. `badId` is returned if there is no such item
equipped on the client.

### Request

    action: unequip
    id: <item's id>

## Use

Use an object by given id. In general object can be used if it is contained in
the player's inventory or equipped onto player. Objects could also 

### Request

    action: use
    id: <item's id>

Optional data for various items:

    x: <global map space x coordinate of target point>
    y: <global map space y coordinate of target point>
    ammoId: <id of an ammo to use with this weapon>

This section is to be completed.

### Response

    result: [ok, badId, badSlot, badAmmoId, badPos]
    message: <text describing what's happened as a result of usage>

# Testing

There are a number of request messages available only when server is in the
testing stage. Such messages are marked with "Testing stage only." If such 
message to be sent while testing stage is not active, server MUST respond with
`"result": "badAction"`.

## Start Testing

This request MUST be sent each time at the beginning of testing stage.
Once this message is responded with `"result": "ok"`, it is valid to state
that testing stage is now active.

It is invalid to request Start Testing when testing stage is already active. In
such case request MUST be answered with `"result": "badAction"`.

### Request

    action: startTesting

### Response

    result: [ok, badAction]

## Stop Testing

Testing stage only.

Each testing stage MUST be closed with this request. Once responded with
`"result": "ok"` it is valid to state that server is no more in the testing
stage.

### Request

    action: stopTesting

### Response

    result: [ok, badAction]

## Set Up Constants

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
    "screenColumnCount": 9,
    "pickUpRadius": 1.5
}
```

### Request

```
action: setUpConst
playerVelocity: <portion of tile player travels through the world per second>
slideThreshold: <portion of tile which gets ignored when moving towards it>
ticksPerSecond: <number of simulation cycles per second>
screenRowCount: <number of tile rows in a rectangle get via `look`>
screenColumnCount: <number of tile columns in a rectangle get via `look`>
pickUpRadius: <maximum distance between player's center and item's one for latter to be picked up>
```

### Response

    result: [ok, badAction]

## Get Constants

Get a set of current constant values.

### Request

    action: getConst

### Response

```
result: ok
playerVelocity: <value>
slideThreshold: <value>
ticksPerSecond: <value>
screenRowCount: <value>
screenColumnCount: <value>
pickUpRadius: <value>
```

## Set Up Map

Testing stage only.

Upload map to server and set it up as currently active. If server assumes
uploaded map to be invalid (e.g. malformed map array or wrong cell value) then
the request MUST be responded with `badMap`. Innermost values of `map` key are
those found in [Dictionary](#get-dictionary).

### Request

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

### Response

    result: [ok, badMap, badAction]

# Data Invariants

This section describes allowable values for data involved.

All units in game are measured in tiles, assuming one tile width and height to
be 1.0. Upscaling this for rendering is up to client.

## Game Objects

Game Objects are the contents of `actors` array of `look` response. Items in
inventory of creatures are also game objects. Hovewer those don't appear in
`look`'s `actors`.

The `type` of game object may be one of:
    - `player`
    - `monster`
    - `item`
    - `projectile`

All MUST have square shape therefore size of actor stands for width == height.
Player's and monster's size MUST be 1. Item's and projectile's size MUST be 0.

### Player

#### Slots

//WIELD - weapon
//BOW - ranged missle launcher
//WIELD & BOW are merged into WEAPON
WEAPON
LEFT - ring
RIGHT - ring
NECK - amulet
//LIGHT - light source
BODY - armor
//OUTER - cloak
ARM - shield
HEAD - helmet
HANDS - gloves
FEET - boots

### Monster

### Item

Every item MUST have a weight. Number of items in inventory of any Creature
(Monster or Player) MUST be limited by maximum weight Creature can handle.

If weight of item to be picked up exceeds maximum Player's weight then pickUp
request MUST result in `tooHeavy`.

If total weight of items in inventory of any Creature somehow exceeds its
carrying capacity then Creature becomes immobilized. If Player is to be
immobilized then `move` request MUST result in `tooHeavy`.

### Projectile