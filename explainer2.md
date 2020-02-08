# Keyboard Lock

`navigator.keyboard.lock()`

## Problem description

(_from the [original explainer](https://github.com/WICG/keyboard-lock/blob/gh-pages/explainer.md)_)

Richly interactive web sites, games and remote desktop/application streaming experiences want to provide
an immersive, full screen experience.
Sites need access to special keys and keyboard shortcuts in full
screen, such as Escape, Alt+Tab, Cmd+`, Ctrl+N, for easily or efficiently navigating through windows,
tabs, applications, menus and gaming functionality.
Without access to these keys, it can be challending for developers to embrace the web for these types of applications.

## Problem description (addendum)

The current Lock API allows developers to specify a set of keys, but if no keys are specified it will by default
capture all keys.
Feedback from developers indicates that they would prefer the default to capture only the Browser keys.
This document describes our proposal for updateing the Lock API.

## Background

There are two groups of keys that can be captured, based on where they are currently being handled.

* **Browser Keys** are keys that are currently handled by the user agent.
Examples of browser keys are Ctrl-S or Ctrl-T.

* **System Keys** are keys that are handled by the underlying operating sytesm.
Examples of System Keys are Alt-Tab (Windows) and Command-Tab (Mac).

## Use Cases

* Remote access where all Browser and System keys are sent to the remote machine. E.g., the Escape key is
sent so that a developer can effectively use VI on the remote machine.
* Remote access where browser keys are sent to the remote machine, but the user can still use OS shortcuts
to switch between virtual desktops locally.
* Playing a game via a streaming service, where the keyboard shortcuts were determined assuming that the
game is running as a native app. E.g., using Escape to pull up the game menu, or players can use ctrl as a
modifer for WASD movement keys.
* Playing a game via a streaming service, where browser shortcuts are used in-game, but the player can
Alt-Tab to quickly switch to another (e.g., a chat) application.
* Playing a web-based fullscreen game that makes use of Alt-Tab in game without switching away from the game.

## What stays the same

Almost everything.

Fullscreen must be active for the API. Interaction with Pointer Lock and the Escape key does not change.
Two methods: `lock()` and `unlock()`. The `lock()` method returns a Promise.

## What changes with this proposal

The current API defaults to capturing all keys (Browser and System). The proposal changes the default
to just capture Browser Keys.

## How the API currently works

The `lock()` method accepts an array of DomKeys to lock/capture. These keys can be a mix of browser and
system level keys. If an array is not speficied, then all keys (browser and system) are captured

`lock()` returns a promise which either resolves w/o a return value or rejects with an error.

Subsequent calls to `lock()` override the previous parameters passed.
For example, if `lock('KeyA')` was called and then `lock('KeyB')` was called, only `KeyB` would be locked.

## The Proposal

Modify the `lock()` so that

* 'lock` With no arguments means the developer is only requesting browser keys
(cf. the current API which captures both browser and system keys).
* `lock` with arguments means the developer wants all browser keys + the specified system keys.

Note that support for capturing system keys requires that the browser install a OS-level keyboard hook.
Browser implementation may choose to only support browser keys (ie, with no arguments).
In this situation, if arguments were passed then the `lock()` call would reject the Promise.

The method signature stays the same, but the behavior changes.

## Benefits of Proposed Changes

While only Chromium-based browsers currently implement this API, making Browser Keys the default simplifies
the minimum implementation requirements.

Supporting Browser Keys requires only changes to the user agent's pipeline, whereas supporting System Keys
requires that the user agent registers a OS level input hook to get access to these keys.

This will allow us to split the specification into 2 levels:

* Level 1 support : browser keys only
* Level 2 support : browser keys + system keys

While no other browser has expressed interest in supporting this
API, simplifying the implementation requirements makes it more likely to be considered. We are actively
seeking feedback from other browser vendors on this matter.

## Impact and backward compatibility

There are only a few users of the API currently, so making this change shouldn't be a large burden.

The common use case is for applicaitons that only want to capture browser keys. This change makes it
easier for this use case, but current users will want to update their calls to remove the argument
to `lock()`. Failure to do so will simply mean that they are capturing more keys than they require.

The only current users who will be "broken" by this change are those that need both browser and 
system keys. This is the more rare use case (typically for remote access or remote shell application)
and transitioning these applications. These apps will need to specify a list of the keys that they
wish to capture. However, these used can update their calls now (with the current API) so that they
are not impacted when the updated version rolls out.

## Alternatives considered

Instead of changing the behavior of the existing API, we also considered:

### Adding new methods to the API

We could add new method(s) (like `lockBrowserKeys()` and `lockAllKeys()`) instead of changing the implementation
of the `lock()` method.

This would make feature detection (to detect old vs. new version) easier, but it makes it less clear what
should happen when the old and new APIs are used at the same time.

### Adding version info

Alternately we could add a version number somewhere (e.g., on `navigator.keyboard`) that could be queried to
determine if `lock()` was the old or new version. This would make it easy to distinguish the old vs new
version, but this is not a typical pattern for web platform APIs.

In the end, we felt that it was better to propose simply fixing the API rather than making the API
description more complicated than it needs to be.

## Privacy & Security

Privacy and security considerations are covered in the current [draft specification](https://wicg.github.io/keyboard-lock/).
The changes proposed here do not change any of the assumptions that would impact privacy or security.
