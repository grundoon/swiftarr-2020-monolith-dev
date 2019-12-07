# `swiftarr`

<p align="center">
    <br>
    <a href="LICENSE">
        <img src="http://img.shields.io/badge/license-MIT-brightgreen.svg" alt="MIT License">
    </a>
    <a href="https://swift.org">
        <img src="http://img.shields.io/badge/swift-5.1-brightgreen.svg" alt="Swift 5.1">
    </a>
</p>
<br>

If you're here for something other than curiosity or to provide [feedback](https://github.com/grundoon/swiftarr/issues),
you're probably developing an API client or maybe even thinking about
[getting involved](https://github.com/grundoon/swiftarr/blob/master/CONTRIBUTING.md) with `swiftarr` development itself.

`swiftarr` is the back-end engine of Twit-arr, implementing API v3. `swiftarr` is unsurprisingly written entirely in
Swift, using the asynchronous Vapor 3 framework built on SwiftNIO, and the SwiftGD wrapper for image
processing. `swiftarr` runs on either macOS or Linux (Ubuntu).

--- 

## Welcome to API v3

* Users
    - can create unlimited sub-accounts, all tied to one registration code
    - can change username
    - can block other users (applies to all sub-accounts) throughout the platform
    - each individual account can mute public content based on individual account or keyword
    - can create unlimited lists of other users – aka: barrels of (sea)monkeys
* Authentication
    - all authentication is done through HTTP headers
    - password recovery uses a a per-user (covers all sub-accounts) recovery key generated when the primary user
account is created (the key is 3 words, nothing cryptic)
* Clients
    - are a new class of account permitted to request certain bulk data
    - are static across all client instances (e.g. RainbowMonkey is a client and all instances use a hard-coded username/password)
    - proxy for the actual client user via an `x-swiftarr-user` header, so that blocks are respected
* Barrels
    - are multi-purpose containers which can hold an array of IDs and/or a dictionary of strings
    - are used for block/mute/keyword lists, user-created lists, things not yet thought of
    - a user's set of seamonkey barrels can be used for filtering or as "auto-complete" results
* Forums
    - all forums belong to a category
    - a category can be restricted ("official") or not (users can create forums therein)
    - categories are created, and forums may be re-categorized, by Moderators
* Events
    - each event on the schedule can have an "official" associated forum
* More
    - soon™

---

## API Client Notes

* Dates are UTC and relative to epoch.
* All authentication is done through HTTP headers. See `AuthController`.
* All entities are referenced by ID (either UUID or Int). While usernames are unlikely to change often, they should be
considered ephemeral and only ever be used to *attempt* to obtain an ID  (`api/v3/users/find/username`).
* Images are the WIP that got shoved aside to push this out, very stubby at the moment.
* Required HTTP payload structs can be encoded as either JSON or MultiPart.
* Returned data is always JSON.
* All endpoints are currently GET/POST to maintain high compatibility for web clients. PUT/DELETE can be mapped
onto them if this offends sensibilities, er, I mean... is desired.
* Query parameters are pretty much avoided in favor of endpoints except for the few necessary cases.
* The intended on-boarding flow:
    1. "welcome!"
    2. username/password ---> recovery key
    3. the user is **strongly** encouraged to screenshot/notes app/write down recovery key before proceeding
    4. "user can now edit profile, read twitarr or forums – if they want to *create* content and Agree to Stuff" then...
    5. registration code --- Basic auth ---> token (user is now logged in)
    6. use token in Bearer auth until token no longer works
* When a token no longer works (401 error response):
    1. try `POST /api/v3/auth/login` with Basic auth
    2. either a new token or a 401 or 403  error is returned
    3. if 401, client is not supplying correct credentials, have user re-enter manually or try recovery
    4. if 403, stop, do not try again, user is read-only    
* more
* soon™

## Getting Started

### Requirements

There are five requirements for either running an instance of `swiftarr` or development itself.

* the [`libgd`](http://libgd.github.io) library
* a recent [Swift](https://swift.org) toolchain (recommend 5.1.x or later)
* the [Vapor](http://vapor.codes) framework
* an instance of [PostgreSQL](https://www.postgresql.org)
* an instance of [Redis](https://redis.io)

A recent version of [Xcode](https://apps.apple.com/us/app/xcode/id497799835?mt=12) is also required for
development. (This is not strictly necessary, but if you walk your own Swift development path, add
"self-sufficiency" to the list.)

### Quickstart - macOS

If running on macOS with Xcode installed, the easiest way to get an instance up and running is via
[Docker](https://www.docker.com/products/docker-desktop).

1. Install Docker.
2. Install `libgd`.
    - On macOS using [Homebrew](https://brew.sh), simply `brew install gd`.
    - On linux, `apt-get libgd-dev` as root.
3. Install the [Vapor](http://docs.vapor.codes/3.0/install/macos/) toolbox. (I don't actually know if this is necessary
for just running an instance, and have no way to easily test that at the moment, but it *might* be needed to get the
correct SSL library and shimming for SwiftNIO.)
4. Download or clone the `switarr` [repository](https://github.com/grundoon/swiftarr).
5. Run `./scripts/instance.sh up` from the `swiftarr` directory to create and launch postgres and redis
Docker containers.
6. Generate the `swiftarr.xcodeproj` file, then open it in Xcode.

```shell
cd <swiftarr-directory>
swift package generate-xcodeproj
open ./swiftarr.xcodeproj
```

Make sure the selected Scheme is "My Mac" in Xcode, hit Run, and `swiftarr` should shortly be available at http://localhost:8081.
To shut down the Docker containers, `./scripts/instance.sh stop`.

Yes, that's a bunch the first time through and it does take some time. From here on out though, it's just a matter of
pulling the latest updates from the repository and regenerating the .xcodeproj file.

(more to come... )

## Documentation

- **API_cheatsheet.md**: A quick endpoint reference for client development, including payload requirements and
return types.

- **docs/**: A complete API reference generated directly from the documentation markup within the source code,
in navigable HTML format.

- **source code**: The source code itself is, in every sense, the definitive documentation. It is thoroughly
documented, incorporating both formatted Swift Documentation Markup blocks (`///`) and organizational
`MARK`s used to generate the HTML `docs/` pages, as well as in-line comments (`//`) to help clarify flow, function,
and thought process. Maintainers and contributors are requested to adhere to the existing standards, or outright
improve upon them!

The `docs/` are generated using the awesome [`jazzy`](https://github.com/realm/jazzy).

The [Vapor](https://vapor.codes) framework has its own [API documentation](https://api.vapor.codes).
