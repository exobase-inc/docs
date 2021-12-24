---
label: Composable Behavior
layout: default
order: 3
---

# Composable Behavior

Business logic does not need to be composable. It is inherently prone to change that you can't anticipate. This is why we isolate it. Everything else, all the structure that builds up to that feature specific business logic should be as easy as snapping legos together. This is a counter balancing principle to Behavior Isolation. Isolating would tend to make things repeated and disorganized. Building our services in modular components that can be composed together to create robust behavior gives us the best of both.

### Hooks & Radash
Composing behavior is done with Exobase hooks and [radash](https://github.com/rayepps/radash). The hooks Exobase provides are developed with configuration in mind.

```ts Comopsing Hooks
import _ from 'radash'
import { useLambda } from '@exobase/lambda'
import { useExpress } from '@exobase/express'
import { useJsonArgs } from '@exobase/hooks'
import { useKeyAuth } from '@exobase/auth'

_.compose(
  useExpress(),
  useJsonArgs(yup => ({
    name: yup.string().required()
  }))
)

_.compose(
  useLambda(),
  useKeyAuth('**********')
  useJsonArgs(yup => ({
    message: yup.string().default('OK')
  }))
)
```
