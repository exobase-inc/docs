---
label: Errors
layout: default
order: 100
description: Learn the core principles of Exobase code
icon: rocket
---

# Errors

Python has an amazing pattern called decorators. A decorator is any function that takes a function as an argument and returns a function. Hooks are very similar. Hooks have an additional optional configuration step. So, hooks wrap functions and augment behavior.

```ts A Slow Down Hook
import type { Props, ApiFunction } from '@exobase/core'

const hook = (config: { ms: number }) => (func: ApiFunction) => async (props: Props) => {
  await new Promise(res => setTimeout(res, config?.ms ?? 0))
  return await func(props)
}

export const useSlow = hook
```
In this example, for no good reason, we made a hook that waits a configured number of ms before continuing. You'll find hooks in the wild with all different styles. Most Exobase hooks will use `_.partial`.

```ts A Slow Down Hook | Partial
import _ from 'radash'
import type { Props, ApiFunction } from '@exobase/core'

const withSlow = async (config: { ms: number }, func: ApiFunction, props: Props) =>
  await new Promise(res => setTimeout(res, config?.ms ?? 0))
  return await func(props)
}

const useSlow = (config: { ms: number }) => (func: ApiFunction) => {
  return _.partial(withSlow, config, func)
}
```

## Two Kinds of Hooks
There are two kinds of hooks. Our `useSlow` hook is your average hook. There is another special hoook called a root hook. `useSlow` expects `Props` as the input to the function, it assumes that something else in the function chain has done the job of converting the framework specific invokation and environment data into `Props`. This is what a root hook does. It assumes it is first, so it should expect framework specific arguments and initialize the `Props` to call the function chain with.

```ts A Root Hook | Pseudocode
import _ from 'radash'
import { initProps, responseFromError, responseFromResult } from '@exobase/core'
import type { Props, Request, Response, ApiFunction } from '@exobase/core'

const withRootHook = async (config: { ms: number }, func: ApiFunction, args: any) =>

  // Step 1: Convert args to request. Read headers, body, query string, etc.
  // and create the Exobase standard request object
  const req: Request = { ... }

  // Step 2: Create the props. Exobase gives you a helper that will auto
  // set a default response object.
  const props: Props = initProps(makeReq(reqAfterMiddlware))

  // Step 3: Call the function chain with your props
  const [error, result] = await _.try<any>(func)(props)

  // Step 4: Convert the error or result into a response. Exobase gives you
  // helpers that will build the object for you
  const response: Response = error
    ? responseFromError(error)
    : responseFromResult(result)

  // Step 5: Apply the response to the framework. In Express this means
  // mutating the res with things like `res.json(response.json)`. In AWS 
  // Lambda this means turning the shape of response into the object shape 
  // that AWS Lambda expects in result of an invocation.
  return applyResponse(response)
}

const useRootHook = (config?: any) => (func: ApiFunction) => {
  return _.partial(withRootHook, config, func)
}
```
In this example you'll see that the only code you need to write yourself is step 1 and step 5. This is the root hook contract. Root hooks are responsible for mapping the framework specific arguments to props and mapping the result of a function back to the framework expected response.

#### See the source
[!ref target="blank" text="useLambda"](https://github.com/exobase-inc/exobase-js/blob/56904c2be5e49d7452b7171ba2fbb86e1e9cc766/packages/lambda/src/useLambda.ts#L18)
[!ref target="blank" text="useExpress"](https://github.com/exobase-inc/exobase-js/blob/master/packages/express/src/useExpress.ts#L35)

## What Do Hooks Do?
A root hook is only responsible for populating the `request` and `response` attributes on the `Props`. Hooks typically do things that populate the remaining attributes: `args`, `auth`, and `services`.

### Args
Some hooks do argument validation. As a practice, we only move data from `request.body` to `args` once it has been validated. This way, an endpoint can access `props.args` with the assumption that it is valid.

As an example, let's make a hook that populates a `version` arg in the props. Maybe your building a large API that changes over time and all request should include a `version`.

```ts Version Arg Hook
import _ from 'radash'
import type { Props, ApiFunction } from '@exobase/core'
import { errors } from '@exobase/core'

const withVersionArg = async (func: ApiFunction, props: Props) =>
  const version = props.body['version']
  if (!version) {
    throw errors.badRequest({
      details: 'Version was not provided in body. Version is required',
      key: 'err.use-version-args.missing'
    })
  }
  if (!isValidVersion(version)) {
    throw errors.badRequest({
      details: 'The version provided is not valid',
      key: 'err.use-version-args.invalid'
    })
  }
  return await func({
    ...props,
    args: {
      ...props.args,
      version
    }
  })
}

const useVersionArg = () => (func: ApiFunction) => {
  return _.partial(withVersionArg, func)
}
```


!!!
Notice `const useVersionArg = () => (func: ApiFunction)` still has the configuration function phase (`= () =>`) even though this hook doesn't take any configuration. As a practice, prefer to leave this here when it is blank.
!!!

You would use your hook when you compose an endpoint:
```ts Using the useVersionArgs Hook
import _ from 'radash'

export const addProduct = async ({ args }: Props) => {
  const version = args.version
  // does add product things...
}

export default _.compose(
  useLambda(),
  useVersionArg(),
  addProduct
)
```

#### See the source
[!ref target="blank" text="useJsonArgs"](https://github.com/exobase-inc/exobase-js/blob/master/packages/hooks/src/useValidation.ts)

### Auth
Your service could use JWT tokens, API Keys, session cookies, or any custom form of authentication and authorization. Exobase provides a package [@exobase/auth](https://github.com/exobase-inc/exobase-js/blob/master/packages/auth) that has some boiler plate auth handled for you. Just like args, we don't want endpoint functions to read directly from the request. Instead, after validating inputs, we set auth information on the `auth` attribute of the props where the endpoint function can access it, gaurnteed that it has been validated.

```ts Api Key Auth Hook
import _ from 'radash'
import type { Props, ApiFunction } from '@exobase/core'
import { errors } from '@exobase/core'


type PropsGetter <T> = (props: exo.Props) => Promise<T>

export async function withApiKey(func: exo.ApiFunction, key: string, props: exo.Props) {
  const header = props.req.headers['x-api-key'] as string

  if (!header) {
    throw exo.errors.unauthorized({
      details: 'This function requires an api key',
      key: 'err.use-api-key.missing'
    })
  }

  const providedKey = header.startsWith('Key ') && header.replace('Key ', '')

  if (!key || !providedKey || providedKey !== key) {
    throw exo.errors.unauthorized({
      details: 'Invalid api key',
      key: 'err.use-api-key.invalid'
    })
  }

  return await func(props)
}

export const useApiKey = (key: string) => (func: ApiFunction) => {
    return _.partial(withApiKey, func, key)
}
```

#### See the source
[!ref target="blank" text="useApiKeyAuthentication"](https://github.com/exobase-inc/exobase-js/blob/master/packages/auth/src/key.ts)
[!ref target="blank" text="useBasicAuthentication"](https://github.com/exobase-inc/exobase-js/blob/master/packages/auth/src/basic.ts)
[!ref target="blank" text="useTokenAuthentication"](https://github.com/exobase-inc/exobase-js/blob/master/packages/auth/src/bearer.ts)

### Services
One of the core Exobase principles is [testability](../core-principles/testable.md). Although there are tools out there that can do complex mocking and replace real instances with mocks at runtime... wouldn't we all just rather not? Aside from the ease of testing, services make it declarative what our endpoint function requires to work (i.e. a database, a cache, a file system, etc.). When testing, its a simple as passing your mocks inside the `Props` when you call your function.

!!! Dependency Injection?
Some have said this is dependency injection. In a way, it is. Dependency injection, by design, solves a much different problem. Dependency injection is a tool used to achieve inversion of control. The simple idea being, the _think_ using another _think_ never knows what the _actual_ instance or type of the _other thing_ is. This lets you swap out instances and types (using dependency injection) to get different behavior. Exobase services are not in any way trying to acheive that. Our goal with services is testability and clarity.
!!!

Here is a very simple hook that comes with the [`@exobase/hooks`](https://github.com/exobase-inc/exobase-js/tree/master/packages/hooks) package to help you add services to your props.

```ts useServices
import _ from 'radash'
import type { Props, ApiFunction } from '@exobase/core'

const withServices = async (func: ApiFunction, services: Record<string, any>, props: Props) => {
  return await func({
    ...props,
    services: {
      ...props.services,
      ...services
    }
  })
}

export const useService = <TServices = Record<string, any>> (services: TServices) => (func: ApiFunction) => {
  return _.partial(withServices, func, services)
}
```
This is a hook, you will likely use on every single one of your functions.
```ts Add Product
import _ from 'radash'

type Args = {
  version: string
}

type Services = {
  db: Database
}

export const addProduct = async ({ args, services }: Props<Args, Services>) => {
  const { version } = args
  const { db } = services
  // does add product things...
}

export default _.compose(
  useLambda(),
  useVersionArg(),
  useServices<Services>({
    db: new Database()
  }),
  addProduct
)
```

#### See the source
[!ref target="blank" text="useServices"](https://github.com/exobase-inc/exobase-js/blob/master/packages/hooks/src/useServices.ts)