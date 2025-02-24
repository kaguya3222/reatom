---
layout: ../../layouts/Layout.astro
title: hooks
description: Reatom for hooks
---

> included in [@reatom/framework](/packages/framework)

All atoms and actions have a hooks to they lifecycle, this package expose friendly helpers to use this hooks.

## `withInit`

Operator to set state creator callback to an atom, which is called by first atom subscription (during transaction).

```ts
import { atom } from '@reatom/core'
import { withInit } from '@reatom/hooks'

const dateAtom = atom(0).pipe(withInit(() => Date.now()))
```

## `onConnect`

Subscribe to atom subscription, optionally return cleanup callback (called during effect queue).

Passed `ctx` have `isConnected` method which checks the passed atom current status.

```ts
import { onConnect } from '@reatom/hooks'

const dispose = onConnect(messagesAtom, (ctx) => {
  const cb = (message) => {
    messagesAtom(ctx, (messages) => [...messages, message])
  }

  WS.on('message', cb)

  return () => WS.off('message', cb)
})
// use `dispose` to remove the hook
```

## `onDisconnect`

Shortcut to `onConnect` return callback.

## `onUpdate`

Derive atom or action update during transaction.

```ts
import { onUpdate } from '@reatom/hooks'

const unlink = onUpdate(pagingAtom, (ctx, page) => fetchData(ctx, page))
// use `unlink` to dispose the hook
```

Very simplified example of lazy analytics connection.

```ts
// analytics.ts
import { isAtom } from '@reatom/core'
import { onUpdate } from '@reatom/hooks'
import * as moduleA from '~/module-a'
// ...
import * as moduleN from '~/module-N'

for (const mod of [moduleA, moduleN]) {
  for (const name of Object.keys(mod)) {
    if (isAtom(mod[name])) {
      onUpdate(mod[name], (ctx, data) => analyticsService.send(name, data))
    }
  }
}
```

## `spyChange`

Spy an atom or an action change in the atom reducer. The difference with [onUpdate](#onupdate) is that `spyChange` is a warm link - works inside atom only when it have a connections.

`spyChange(CtxSpy, anAtom, (value) => any): isChanged`

```ts
import { atom } from '@reatom/core'
import { spyChange } from '@reatom/hooks'

export const someAtom = atom((ctx, state = initState) => {
  spyChange(ctx, someAction, (payload) => {
    state = state + payload
  })
  // OR
  if (spyChange(ctx, someAction)) {
    state = state + payload
  }
})
```

<!-- ## `controlConnection` -->

<!-- ## `isConnected` -->
