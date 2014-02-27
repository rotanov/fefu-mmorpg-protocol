## FEFU MMORPG Protocol — FEMP/0.1

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

### Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119).

An implementation is not compliant if it fails to satisfy one or more
of the MUST or REQUIRED level requirements for the protocols it
implements. An implementation that satisfies all the MUST or REQUIRED
level and all the SHOULD level requirements for its protocols is said
to be "unconditionally compliant"; one that satisfies all the MUST
level requirements but not all the SHOULD level requirements for its
protocols is said to be "conditionally compliant."

### Introduction

Client and server communicate with request and response messages.
Each message MUST be represented by a single JSON object.
Messages are sent via either HTTP/1.1 or WebSocket protocol.

Request message MUST contain a single "action" name with a corresponding string
value determining required action.
Each request message MUST be answered with a corresponging response message.
Response message MUST contain a single "result" name with a corresponding value
describing result.
Both request and response messages MAY contain any other name/value pairs
specific for particular request/response.

### Terminology

TBD — http://en.wiktionary.org/wiki/TBD

### Authorization

An authorization stage of communication MUST use HTTP/1.1 as an underlying
transport. A method of HTTP request MUST be POST. Message MUST be HTTP
message-body. Content-Type header of HTTP response MUST be "application/json".

#### Register

##### Request

    action: register
    login: <new user's login> 
    password: <new user's password>

##### Response

    result: [ok, badPassword, badLogin, loginExists]

#### Login

##### Request
 
    action: login
    login: <user's login> 
    password: <user's password>

##### Response

    result: [ok, invalidCredentials]
    sid: <string representation of session identifier>
    webSocket: <TBD>

#### Logout

##### Request

    action: logout
    sid: <user's sid>

##### Response

    result: [ok, badSid]
