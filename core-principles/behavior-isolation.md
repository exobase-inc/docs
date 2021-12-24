---
label: Behavior Isolation
layout: default
order: 4
---

# Behavior Isolation

One of the most important qualities of nimble software is to have the busines logic isolated from the peripheral and supporting code. This is key for both change and understanding. We've all opened a 600 line controller or endpoint and spent an hour reading it up and down to get an idea of what it does. The exobase framework provides a pattern to isolate this external logic: hooks. Using hooks with Exobase gives you a way to isolate peices of logic that are not business specific outside of your endpoint function. Common tasks like request parsing, validation, and authorization can be setup and configured at the site of your function without getting involved in the actual endpoint.

```ts Example: Tweet Endpoint
import _ from 'radash'
import type { Props } from '@exobase/core'
import { useLambda } from '@exobase/lambda'
import { useJsonArgs } from '@exobase/hooks'
import { useTokenAuth } from '@exobase/auth'

type Args = {
  message: string
}

export const postTweetEndpoint = async (props: Props<Args>): Promise<Tweet> => {
  const userId = props.auth.token.sub
  const message = props.args.message
  // create & return tweet
}

export default _.compose(
  useLambda(),
  useTokenAuth({
    secret: '***********',
    type: 'id'
  }),
  useJsonArgs<Args>(yup => ({
    message: yup.string().required().max(160)
  }),
  postTweetEndpoint
)
```

Notice specifically how we don't check that the token, user id, or message input is valid inside of the `postTweetEndpoint` function. We can assume that if the hooks allowed the function chain to continue that they did their job of validating the input.
