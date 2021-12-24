---
label: Framework Agnostic
layout: default
order: 5
---

# Framework Agnostic

In order to keep our services flexible we can't allow them to know about the underlying framework that is running it. We don't want to have to rewrite our service because we move it from a GCP Cloud Function invoked with Express to an AWS Lambda function invoked with an event/context. We've defined a request/response interface that modules running Exobase reference. An object containing all the data involved in the request is provided in a props object.

```ts Exobase Request / Response
export type Request = {
    headers: Record<string, string | string[]>
    url: string
    body: Record<string, any> | string | null
    cookies: Record<string, string>
    method: string
    query: Record<string, string>
}

export type Response = {
    _type: 'exobase.response'
    _rid: string
    headers: Record<string, string | string[]>
    status: number
    json: any
    cookies: string[]
}

export interface Props <
    ArgType = any, 
    ServiceType = any,
    AuthType = any
> {
    auth: AuthType
    args: ArgType
    services: ServiceType
    req: Request
    response: Response
}
```

#### See the source
[!ref target="blank" text="@exobase/core"](https://github.com/exobase-inc/exobase-js/blob/master/packages/core/src/types.ts)


```ts Example Function
import _ from 'radash'
import { useLambda } from '@exobase/lambda'
import type { Props } from '@exobase/core'

export const healthCheckEndpoint = (props: Props) => {
  return { message: 'pong' }
}

export default _.compose(
  useLambda(),
  healthCheckEndpoint
)
```

Exobase hooks like `useJsonArgs` and `useService` expect `Props` as input to their functions and return a function that expects `Props`. Root hooks like `useLambda` and `useExpress` are responsible for mapping the underlying framework into `Props`, calling the function chain, and mapping the result back to the underlying framework.

#### See the source
[!ref target="blank" text="@exobase/lambda"](https://github.com/exobase-inc/exobase-js/blob/master/packages/lambda/src/useLambda.ts#L18)

[!ref target="blank" text="@exobase/express"](https://github.com/exobase-inc/exobase-js/blob/master/packages/express/src/useExpress.ts#L35)

