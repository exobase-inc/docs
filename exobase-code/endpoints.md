# Endpoints

Endpoints are a composition of hooks, where each hook is optionally configured to provide the behavior wanted for the specific endpoint. This allows you to boil your endpoint function down to business logic only, moving everything else into hooks where it can be reused and configured for different behavior in other endpoints.

By definition, an endpoint function is any function that takes in props and returns some result. This is an endpoint function:

```ts
import type { Props } from '@exobase/core'

export const pingEndpoint = (props: Props) => {
  return {
    message: 'pong'
  }
}
```

This function isn't very useful. If we want to call it over a network we'll need a root hook to do the framework translation. Let's assume we want to host our function on AWS Lambda.

```ts
import _ from 'radash'
import type { Props } from '@exobase/core'
import { useLambda } from '@exobase/lambda'

export const pingEndpoint = (props: Props) => {
  return {
    message: 'pong'
  }
}

export default _.compose(
  useLambda(),
  pingEndpoint
)
```

## Open Source Examples
The best way to get to know this pattern is to read it in real life. Here are some links to the Exobase platform's open source API endpoints.

[!ref target="blank" text="/health/ping"](https://github.com/exobase-inc/exobase-api/blob/master/src/modules/health/ping.ts)
[!ref target="blank" text="/platforms/getById"](https://github.com/exobase-inc/exobase-api/blob/master/src/modules/platforms/getById.ts#L1)