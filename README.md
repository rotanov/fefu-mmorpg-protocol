# FEFU MMORPG Protocol — FEMP/0.4

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
    - [Login](#login)
    - [Logout](#logout)
- [Game Interaction](#game-interaction)
    - [Common Invariants](#common-invariants)
    - [Destroy Item](#destroy-item)
    - [Drop](#drop)
    - [Equip](#equip)
    - [Examine](#examine)
        - [Single id Examine](#single-id-examine)
        - [Map Cell Examine](#map-cell-examine)
        - [Multiple id Examine](#multiple-id-examine)
    - [Get Dictionary](#get-dictionary)
    - [Logout](#logout-1)
    - [Look](#look)
    - [Move](#move)
    - [Pick Up](#pick-up)
    - [Tick](#tick)
        - [Possible Events](#possible-events)
            - [Attack](#attack)
            - [Effect](#effect)
    - [Unequip](#unequip)
    - [Use](#use)
- [Testing](#testing)
    - [Get Constants](#get-constants)
    - [Put Item](#put-item)
    - [Put Mob](#put-mob)
    - [Put Player](#put-player)
    - [Enforce](#enforce)
    - [Set Location](#set-location)
    - [Set Up Constants](#set-up-constants)
    - [Set Up Map](#set-up-map)
    - [Start Testing](#start-testing)
    - [Stop Testing](#stop-testing)
- [Data Invariants](#data-invariants)
    - [Game Objects](#game-objects)
    - [Player](#player)
        - [Slots](#slots)
    - [Monster](#monster)
    - [Item](#item)
        - [Item Description](#item-description)
            - [Item Class](#item-class)
            - [Item Type](#item-type)
            - [Item Subtype](#item-subtype)
            - [Bonus description](#bonus-description)
            - [Effect description](#effect-description)
                - [Ongoing Effect Description](#ongoing-effect-description)
                - [Bonus Effect Description](#bonus-effect-description)
    - [Projectile](#projectile)
    - [Race](#race)
    - [Flags](#flags)
    - [Stats](#stats)

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

Key in context of JSON message stands for name in name/value pair which is the
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

    result: one of: ok, badPassword, badLogin, loginExists

## Login

### Request
 
    action: login
    login: <client's login> 
    password: <client's password>

### Response

    result: one of: ok, invalidCredentials
    sid: <string representation of session identifier>
    webSocket: <WebSocket server URI>
    id: <player ID for use with Game Interaction requests>
    fistId: <fists' id for use it when no weapon equipped>

## Logout

### Request

    action: logout
    sid: <client's sid>

### Response

    result: one of: ok, badSid

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

## Destroy Item

Destroys an item. It is possible if distance between item and player centers
doesn't exceed pickUpRadius or item is already in player's inventory. Otherwise
result MUST be `badId`. If object with this id is not an item result MUST be
`badId`.

If item has `consumable` class or `expendable` type (see [Items](#items))
request body MUST contain `amount` field that specify amount of destroyed items.

When `amount` field contain negative value or value that is greater than count of
items in stack or destroyed item is not stackable, result MUST be `badAmount`.

### Request

    action: destroyItem
    id: <item's id>
    amount: <items amount>

## Drop

Drop item from inventory to the ground. Item's x and y now MUST be the same as
client's player ones.

If dropped item has `consumable` class or `expendable` type (see [Items](#items))
request body MUST contain `amount` field that specify amount of dropped items.

When `amount` field contain negative value or value that is greater than count of
items in stack or dropped item is not stackable, result MUST be `badAmount`.

### Request

    action: drop
    id: <id>
    amount: <dropped items amount>

### Response

    result: one of: ok, badSid, badId, badAmount

## Equip

Equips item of given id onto specified slot. If it is impossible to equip this
item onto that slot badSlot is returned. If slot is already occupied then item
which is already there gets automatically unequipped and takes of the item being
equipped.

If item is on the ground and it is possible to Pick Up it with Pick Up request
then it is also possible to try to equip this item from the ground. If the
result of such equipping is `ok` than item is automatically gets picked up.

When equipping a `weapon` of subtype `two-handed` or `bow` while wielding a
some item in other hand — last one MUST be unequipped automatically.

If equipping has modified contents of other slots, then response MUST
contain `slots` field, that is mapping from modified slot to id of item, that now
is equipped in this slot.

### Request

    action: equip
    id: <item's id>
    slot: <slot to equip onto>

### Response

    slots: { <slot> : <item id>, ... }
    result: one of: ok, badId, badSid, badSlot

## Examine

There are three versions of Examine. One is for single id, other is for map
cell and last is OPTIONAL for multiple id's.

### Single id Examine

#### Request

    action: examine
    id: <actor's identifier>

#### Response

    id: <actor's id>
    type: <actor's type. One of `player`, `monster`, `item`, `projectile`>
    x: <x coordinate>
    y: <y coordinate>
    result: one of: ok, badSid, badId

If type is `item` response MUST contain the following:

    item: {<Item Description*>}

See [Item Description](#item-description)

If type is either `player` or `monster` response MAY contain the following:

    health: <actor's current number of health points>
    maxHealth: <actor's maximum number of health points>
    mana: <actor's current number of mana points>
    maxMana: <actor's maximum number of mana points>
    inventory: [<item description>, ...]

Under various circumstances `inventory` field MAY be present or not. However if
given id is an id of a player corresponding to client issued an `Examine`
request and this player's inventory is not empty the field `inventory` MUST
always be present. 

If present, `inventory` is an array of item ids.

If `type` is `monster` response MAY contain these fields:
    
    name: <name of a monster>
    mobType: <string describing the type of a monster>

If `type` is `player` response MAY contain these fields:

    slots: {"<slot name>": <item's id>, ...}
    login: <player's login>

Rules for presence of field `slots` are the same as for field `inventory`.
If slot is empty it MAY be omitted.

For possible slot names see [Slots](#slots)

### Map Cell Examine

#### Request

(x, y) pair belongs to a cell. This cell is about to be examined.

    action: examine
    x: <x coordinate on map>
    y: <y coordinate on map>

#### Response

Response JSON object MUST contain a single field `data` which is an array of
objects each representing a result from single id version of Examine. data MUST
contain information for all objects belonging to given cell.

    data: [{<single-id-examine's result}]

### Multiple id Examine

The following is OPTIONAL. Examine request could be issued to examine multiple
ids.  In such case `id` field of request is an array of item's ids. Then
response is an array of objects each containing fields from Examine response for
single id. e.g.

    ```json
    "data":
    [
        {
            "id": 42,
            "type": "player",
            "login": "John",
            "x": 0,
            "y": 100,
            "result": "ok"
        },
        {
            "id": 43,
            "type": "player",
            "login": "Snow",
            "x": 1,
            "y": 99,
            "result": "ok"
        },
        {
            "result": "badId"
        }
    ]
    ```

## Get Dictionary

Dictionary is a JSON object describing mapping from game map cell (recieved via
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
    mana: <actor's current number of mana points>
    maxMana: <actor's maximum number of mana points>

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
    direction: one of: west, north, east, south
    tick: <tick number for move action>

## Pick Up

Request for client player to pick up an item with given ID. Object with this ID
MUST have type `item` and distance between player's center and item's center
MUST be less or equal than constant pickUpRadius. Otherwise nothing is picked up
and result is `badId`.

If item is contained in someone's inventory than result is `badId`.

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
    slot: <slot to be unequipped>

### Response

    result: one of: ok, badSlot

`badSlot` MUST be returned if `slot` field contain invalid slot specificator
or doesn't exist in request.

## Use

Use an object by given id. In general object can be used if it is contained in
the player's inventory or equipped onto player. Objects can also be used if
laying on the ground and can be picked up with Pick Up request.

Some item require certain additional data to be specified. e.g. if using
equipped weapon client MUST specify `x` and `y`.

`badSlot` MUST be returned if using item required to be equipped e.g. using
weapon present in inventory but unequipped.

If no item equipped as a weapon and player want to attack, `fistId` (see [Login](#login))
MUST be used in request.

### Request

    action: use
    id: <item's id>

Optional data for various items:

    x: <global map space x coordinate of target point>
    y: <global map space y coordinate of target point>

This section is to be completed.

### Response

    message: <text describing what's happened as a result of usage>
    result: one of: ok, badId, badSlot, badPos

# Testing

There are a number of request messages available only when server is in the
testing stage. Such messages are marked with "Testing stage only." If such 
message to be sent while testing stage is not active, server MUST respond with
`"result": "badAction"`.

All testing API provided via web-socket. All testing requests need SID field in
request body and can return `badSid` if SID is invalid.

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

## Put Item 

Testing stage only.

### Request

    action: "putItem"
    x: <x coordinate of item's center>
    y: <y coordinate of item's center>
    item: {<Item Description*>}

See also:

- [Item Description](#item-description)

### Response

    result: ok, badPlacing, badItem
    id: <item's id>

## Put Mob

Testing stage only.

Put specified mob onto the level map.

### Request

    action: "putMob"
    x: <mob's x coordinate>
    y: <mob's y coordinate>
    stats: {<Stats*>}
    inventory : [{<Item Description*>}, ...]
    flags: [<Flags*>, ...]
    race: <Race*>
    dealtDamage: <mob's dealt damage by single attack>

Field `dealtDamage` `MUST` exist in request. This field is specified by
string of the form `NdM`, where N is count of verges dice and M is count 
of dice pops.  

See also:

- [Stats](#stats)
- [Item Description](#item-description)
- [Flags](#flags)
- [Race](#race)

### Response

    id: <mob's id>
    result: one of: ok, badPlacing, badFlag, badRace, badInventory, badStats, badDamage

## Put Player

Testing stage only.

Put player instance onto the level map, specifying inventory, slots and stats.
Player has `CAN_MOVE` and `CAN_BLOW` flags by default.

### Request

    action: "putPlayer"
    x: <player's x coordinate>
    y: <player's y coordinate>
    inventory: [{<Item Description*>}, ...]
    slots: {<Slot name. Slots*> : {<Item Description*>}, ...}
    stats: {<Stats*>}

See also:

- [Stats](#stats)
- [Item Description](#item-description)
- [Slots](#slots)

### Response

    sid: <player's session id>
    id: <player's id>
    fistId: <player's fist id>
    inventory: [<items' id generated in inventory>]
    slots: {<Slot name. Slots*> : <items' id generated for slot>, ...}
    result: one of: ok, badPlacing, badInventory, badSlot, badStats

If there was not items in `inventory`/`slots` field in `putPlayer` request
then corresponding fields in response MAY be omitted.

## Enforce

Testing stage only.

This request tell server perform some player's action.

### Request

    action: "enforce"
    enforcedAction: <object that represent any action of player>

### Response

    actionResult: <object that represent result of perfoming requested action>
    result: one of: ok, badSid, badEnforcedAction

## Set Location

Testing stage only.

Set x, y coordinates of player.

### Request

    action: "setLocation"
    x: <desired player's x coordinate>
    y: <desired player's y coordinate>

### Response

    result: on of: ok, badPlacing, badSid


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

    result: one of: ok, badAction

## Set Up Map

Testing stage only.

Upload map to server and set it up as currently active. If server assumes
uploaded map to be invalid (e.g. malformed map array or wrong cell value) then
the request MUST be responded with `badMap`. Innermost values of `map` key are
those found in [Dictionary](#get-dictionary).

Minimal size of map is 1x1. Empty map is `badMap`.

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

    result: one of: ok, badMap, badAction

## Start Testing

This request MUST be sent each time at the beginning of testing stage.
Once this message is responded with `"result": "ok"`, it is valid to state
that testing stage is now active.

It is invalid to request Start Testing when testing stage is already active. In
such case request MUST be answered with `"result": "badAction"`.

### Request

    action: startTesting

### Response

    result: one of: ok, badAction

## Stop Testing

Testing stage only.

Each testing stage MUST be closed with this request. Once responded with
`"result": "ok"` it is valid to state that server is no more in the testing
stage.

### Request

    action: stopTesting

### Response

    result: one of: ok, badAction

# Data Invariants

This section describes involved data.

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

All of those MUST have square shape. Player's and monster's size MUST be 1.
Item's and projectile's size MUST be 0.

## Player

### Slots

- `left-hand` - one-handed weapon or shield slot
- `right-hand` - one-handed weapon or shield slot
- `ammo` - slot for weapon expendables. e.g. arrows for bow.
- `left-finger` - ring
- `right-finger` - ring
- `neck` - amulet
- `body` - armor
- `head` - helmet
- `forearm` - gloves
- `feet` - boots

## Monster

Monsters have a simple logic realized by flags (see [Flags](#flags)). Describe
main concepts of monsters' behavior logic.

1. If monster found target, it has to go to it.
2. If monster gets attacked by player, it MUST get player as new target.
3. If monster gets attacked by another monster, first one has to options:
    - if it was fighting with another monster, one MAY switch its target to attacker
    - if it wasn't, one MUST gets attacker as new target

## Item

Every item MUST have a weight. Number of items in inventory and slots of any
Creature (Monster or Player) MUST be limited by maximum weight
Creature can handle.

If weight of item to be picked up exceeds maximum Player's weight then pickUp
request MUST result in `tooHeavy`.

If total weight of items in inventory of any Creature somehow exceeds its
carrying capacity then Creature becomes immobilized. If Player is to be
immobilized then `move` request MUST result in `tooHeavy`.

### Item Description

Some requests from Testing section MAY induce a need to describe an item.
Examine returns itemData in the same form.

Item description is a set of JSON fields:

    weight: <item's weight>
    class: <Item Class*>
    type: <Item Type*>
    subtype: <Item Subtype*>
    bonuses : [{<Bonus Description*>}, ...]
    effects : [{<Effect Description*>}, ...]

Based on the context those fields may be wrapped in JSON object.

Bonus application order is determined via order of bonuses elements.
Same applies to effects.

Field `subtype` MUST be omitted if there is no subtypes specified for item type.
Otherwise it MUST be present.

No `shield` can be equipped with `bow` or `two-handed`.
`arrows` can only be equipped with `bow`.

#### Item Class

- `garment`
- `consumable`

#### Item Type

types for `garment` class:

- `amulet`
- `ring`
- `armor`
- `shield`
- `helm`
- `gloves`
- `boots`
- `weapon`
- `expendable`

#### Item Subtype

subtypes for `weapon` type of `garment` class:

- `one-handed`
- `two-handed`
- `bow`

subtypes for `expendable` type of `garment` class:

- `arrows`

#### Bonus description

Only equipment may have bonuses.

TBD: REFORMULATE section

Json-object represent bonus description.
There is a one kind of bonuses in Angband: constant bonus to some stat.
Our team added something like it: bonus to any stat of active object (mob, player),
but it can be calculated as percent bonus. So, we have two kinds of bonuses: constant bonus and percent bonus.
Calculation for constant bonus: <stat> += <bonus_val>
Calculation for percent bonus: <stat> += <stat> * <bonus_val>
Value of bonus can be a negative.

    stat: <stat modified by bonus>
    effectCalculation: <const|percent>
    value: <value of bonus>

#### Effect description

TBD: REFORMULATE section

Json-object respresent effect description.
Effects in Angband are something hazy, so I invented my own.
Effect is something, that modified some only stat during some time.
There is two kinds of effects: OnGoingEffect (modify stat continuously
with some fixed value) and BonusEffect (add some bonus).

##### Ongoing Effect Description

    stat : <stat modified by effect>
    duration : <effect duration in seconds>
    type: "ongoing"
    value: <value of effect>

##### Bonus Effect Description

    duration: <effect duration in seconds>
    effectType: "bonus"
    bonus: <bonus description>

## Projectile

## Race

Both players and monsters have race. Player always have `PLAYER` race.

Possible races are:

- `ORC`
- `EVIL`
- `TROLL`
- `GIANT`
- `DEMON`
- `METAL`
- `DRAGON`
- `UNDEAD`
- `ANIMAL`
- `PLAYER`

Races aree taken from Angband (except for the `PLAYER` race).

## Flags

The following flags are available:

- `CAN_MOVE`
- `CAN_BLOW`
- `HATE_<race>`

Angband has many flags. Of our flags `CAN_MOVE` and `CAN_BLOW` have relation to
Angband's `NEVER_MOVE` and `NEVER_BLOW` ones.

TBD: REFORMULATE the following:
    If mob hasn't flag NEVER_MOVE (NEVER_BLOW), it has CAN_MOVE (CAN_BLOW)
    flag by default. Otherwise, if mob has never-flag in angband, it hasn't
    corresponding flag in our MMO.

Angband have no `HATE_<race>` flag. E.g. `HATE_ORC`, `HATE_PLAYER` etc.

TBD: describe thoroughly what does it mean to `HATE_<race>` etc

## Stats

Both players and monsters have stats. Stat describes to what extent a character
possesses a natural, in-born characteristic.

Stats in Angband:

- STR
- INT
- WIS
- DEX
- CON
- SPEED
- STEALTH
- SEARCH
- LIGHT
- TUNNEL

Our game does not implement all of those. Instead we are using the following
stats:

- `STRENGTH` (STR)
- `INTELLIGENCE` (INT)
- `DEXTERITY` (DEX)
- `SPEED` (SPEED)
- `DEFENSE`
- `MAGIC_RESISTANCE`
- `CAPACITY`
- `HP`
- `MAX_HP`
- `MP`
- `MAX_MP`

Some requests from Testing section require to specify stats field. Stats field
MUST be JSON object containing zero or more of the aforementioned stats.

If any particular stat as well as the whole stats field of request is not
required for testing it MAY be omitted.

TBD: rework stats and describe which stats affect which.