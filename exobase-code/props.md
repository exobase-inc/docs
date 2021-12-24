---
label: Props
layout: default
order: 100
icon: rocket
---

# Props

To allow for exobase services running on many cloud provider / services we designed a simple interface to represent a request / response anywhere. A root hook is responsible for mapping from the framework it was designed for to the standard exobase `Props` object that contains all the data about the request and response with additional attributes used in other Exobase hooks and endpoints.

```ts The Props Type
type Props <
  ArgType = any, 
  ServiceType = any,
  AuthType = any
> = {
  auth: AuthType
  args: ArgType
  services: ServiceType
  req: Request
  response: Response
}
```
The props object is passed to your endpoint function as the sole argument.

```ts Props Example
import { Props } from '@exobase/core'
const pingEndpoint = async (props: Props) => {
  return { message: 'pong' }
}
```
The props object is created by the root hook. Hooks can pass new `Props` objects on through the function chain to override or alter the `Props` they were passed.


## Request

```ts The Request Type
export type Request = {
  headers: Record<string, string | string[]>
  url: string
  body: Record<string, any> | string | null
  cookies: Record<string, string>
  method: string
  query: Record<string, string>
}
```
This is the request object you'll have access to in your hooks and inside your endpoint. It is small, clean, and simple. All root hooks should provide this request data completely. It can be extended to add additional data when needed for specific cases or frameworks.


## Response

```ts The Response Type
type Response = {
  _type: 'exobase.response'
  _rid: string
  headers: Record<string, string | string[]>
  status: number
  json: any
  cookies: string[]
}
```
This object is more of a helper. For most cases you can forget about it. It is useful when you want to alter the response beyond the body being returned.

```ts Altering the Response
import { Props } from '@exobase/core'
const pingEndpoint = async (props: Props) => {
  return { message: 'pong' }
}
const pingEndpoint = async (props: Props) => {
  return {
    ...props.response,
    headers: {
      'X-Health-Check': 'OK'
    },
    json: {
      message: 'pong'
    }
  }
}
```
Both these functions return a json body of `{ message: 'pong' }`. The second one also returns a `X-Health-Check` header. You can see here, you can extend the provided base response and return it when needed to get more fine grain control over the http response.

#### See the source
[!ref target="blank" text="@exobase/core"](https://github.com/exobase-inc/exobase-js/blob/master/packages/core/src/types.ts#L80)