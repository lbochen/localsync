[![NPM](https://raw.githubusercontent.com/noderaider/localsync/master/public/images/localsync.gif)](https://npmjs.com/packages/localsync)

### serversync

**a lightweight module to sync JS objects in realtime across tabs / windows of a browser *SERVER VERSION*.**

#### Features

* Uses local storage event emitters to sync objects in realtime across tabs.
* Never calls the tab that the event occurred on.
* Falls back to cookie polling internally if using an unsupported browser (IE 9+ / Edge).
* Isomorphic.
* Tested with mocha.

[![Build Status](https://travis-ci.org/noderaider/localsync.svg?branch=master)](https://travis-ci.org/noderaider/localsync)
[![codecov](https://codecov.io/gh/noderaider/localsync/branch/master/graph/badge.svg)](https://codecov.io/gh/noderaider/localsync)

[![NPM](https://nodei.co/npm/localsync.png?stars=true&downloads=true)](https://nodei.co/npm/localsync/)

## Install

`npm i -S serversync`


## How to use

```js
import serversync from 'serversync'

/** Create an action that will trigger a sync to other tabs. */
const action = (userID, first, last) => ({ userID, first, last })

/** Create a handler that will run on all tabs that did not trigger the sync. */
const handler = (value, old, url) => {
  console.info(`Another tab at url ${url} switched user from "${old.first} ${old.last}" to "${value.first} ${value.last}".`)
  // do something with value.userID
}

/** Create a synchronizer. serversync supports N number of synchronizers for different things across your app. */
const usersync = serversync('user', action, handler)

/** Start synchronizing. */
usersync.start()

/** IE / Edge do not support local storage across multiple tabs. serversync will automatically fallback to a cookie polling mechanism here. You don't need to do anything else. */
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

```js
serversync(key: string, action: (...args) => payload, handler: payload => {}, [opts: Object]): { start, stop, trigger, isRunning, isFallback }
```

**opts**

**name**    | **type**    | **default**   | **description**
--------    | --------    | -----------   | ---------------
`tracing`   | `boolean`   | `false`       | toggles tracing for debugging purposes
`logger`    | `Object`    | `console`     | the logger object to trace to
`loglevel`  | `string`    | `'info'`      | the log level to use when tracing (`error`, `warn`, `info`, `trace`)

**IE / Edge fallback props for `cookiesync`**

**name**        | **type**      | **default**   | **description**
--------        | --------      | -----------   | ---------------
`pollFrequency` | `number`      | `3000`        | the number in milliseconds that should be used for cookie polling
`idLength`      | `number`      | `8`           | the number of characters to use for tracking the current instance (tab)
`path`          | `string`      | `'/'`         | The path to use for cookies
`secure`        | `boolean`     | `false`       | Whether to set the secure flag on cookies or not (not recommended)
`httpOnly`      | `boolean`     | `false`       | Whether to set the http only flag on cookies or not
