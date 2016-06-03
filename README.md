[![NPM](https://raw.githubusercontent.com/noderaider/localsync/master/public/images/localsync.gif)](https://npmjs.com/packages/localsync)

**a lightweight module to sync JS objects in realtime across tabs / windows of a browser.**

#### Features

* Uses local storage event emitters to sync objects in realtime across tabs.
* Never calls the tab that the event occurred on.
* Falls back to cookie polling internally if using an unsupported browser (IE 9+ / Edge).
* Isomorphic.
* Tested with mocha.

[![Build Status](https://travis-ci.org/noderaider/localsync.svg?branch=master)](https://travis-ci.org/noderaider/localsync)

[![NPM](https://nodei.co/npm/localsync.png?stars=true&downloads=true)](https://nodei.co/npm/localsync/)


## Install

`npm i -S localsync`


## How to use

```js
import localsync from 'localsync'

/** Create an action that will trigger a sync to other tabs. */
const action = (userID, first, last) => ({ userID, first, last })

/** Create a handler that will run on all tabs that did not trigger the sync. */
const handler = (new, old, url) => {
  console.info(`Another tab at url ${url} switched user from "${old.first} ${old.last}" to "${new.first} ${new.last}".`)
  // do something with new.userID
}

/** Create a synchronizer. localsync supports N number of synchronizers for different things across your app. */
const usersync = localsync('user', action, handler)

/** Start synchronizing. */
usersync.start()

/** IE / Edge do not support local storage across multiple tabs. localsync will automatically fallback to a cookie polling mechanism here. You don't need to do anything else. */
if(usersync.isFallback)
  console.warn('browser doesnt support local storage synchronization, falling back to cookie synchronization.')

/** Trigger an action that will get handled on other tabs. */
usersync.trigger(1, 'jimmy', 'john')

setTimeout(() => {
  /** Trigger another action in 5 seconds. */
  usersync.trigger(2, 'jane', 'wonka')
}, 5000)

setTimeout(() => {
  /** If its still running, stop syncing in 10 seconds. */
  if(usersync.isRunning)
    usersync.stop()
}, 10000)
```

## Documentation

`localsync(key: string, action: (...args) => payload, handler: payload => {}, [opts: Object])`

**opts**

name      | default
----      | -------
tracing   | false
logger    | console
loglevel  | 'info'

**IE / Edge fallback props for cookiesync**

name          | default
----          | -------
idLength      | 8
pollFrequency | 3000 (MS)
path          | '/'
secure        | false
httpOnly      | false
