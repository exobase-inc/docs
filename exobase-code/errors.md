# Errors

You can import our errors module from `@exobase/core`. This module can be used to create your own error objects, or just throw existing ones.

```ts
import { errors } from '@exobase/core'

const lostPingError = errors.createError({
  details: 'Your ping has been lost to the darkness',
  message: 'Lost Ping',
  status: 502,
  code: 5021
})

const pingEndpoint = async (props: Props) => {
  if (process.env.HAS_PONG) {
    throw lostPingError({
      key: 'exo.err.health.ping.lopi'
    })
  }
  throw errors.noFound({
    details: 'There is no ping here',
    key: 'exo.err.health.ping.nofo'
  })
}
```

## But Why?
Why does Exobase have its own errors? Its simple, root hooks, developed without any knowledge of each other need a consistent pattern rely on for error handling. Under the hood, an exobase error is simply an object with this shape:

```ts
export interface ApiError {
  name: 'exobase.error'
  message: string
  status: number
  code: number
  key: string
  details: string
}

```

All root hooks, or any other hook that is interested in dealing with errors, can look for this object (identifying it by having `name: 'exobase.error'`) and know that it is an error thrown by the engineer. When a hook catches an error that does not match this shape (again, decided by the presence of the name attribute), it knows to treat it as an unknown error. In this case, it may decide to return a limited amount of information to the user for security.

## Keys
You may have noticed the `key` attribute. This is to help when debugging. Internally at Exobase, it has become a best practice that we wanted others to be able to take advantage of and enjoy. Error keys, when used correctly, allows us to find the location of a logged or reported error and therefore identify the issue extremely quickly.

### How do keys work?
Error keys work by being unique. The work is admitedly put on the engineering team to ensure this. There has been talk of building linter rules that could do the checking, or automated build tools to do the writing of keys but in our experience the effort is low and the reward for having them written in the code in plain site is very high.

When a user reports an error, or we see one in the logs, we copy the key and do a global search in our codebase. Within a few seconds we have identified the exact line where the error was thrown.

### Key rules
If you're going to get the most out of error keys, here's how to do it:
1. They must be unique. We developed a pattern to make it easy and add additiional value to our keys: `exo.err.{module}.{file}.{rand-phrase}`. For example, `exo.err.services.update.nofo`. Lately, we have moved further away from truly random phrases and used shorter abbreviated phrases that help indicate the error. `nofo` is short for not found. Previously, we used constellation names, boat names, and pokemon names. Either way, as long as keys are unique, they will get the job done.
2. They must be specified at the final creation of the error. Don't make a helper function that makes error objects with the keys set. It is tempting, don't go there. The value of the keys will quickly be eliminated. If you open the project, search for a key, but still need to traverse the code to find the actual call site, you have made more work, not less.

#### See the source
[!ref target="blank" text="errors"](https://github.com/exobase-inc/exobase-js/blob/master/packages/core/src/errors.ts)
[!ref target="blank" text="ApiError"](https://github.com/exobase-inc/exobase-js/blob/master/packages/core/src/types.ts#L2)