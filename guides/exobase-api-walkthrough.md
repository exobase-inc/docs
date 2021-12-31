---
label: Exobase API Walkthrough
layout: default
order: 999
---

# Exobase API Walkthrough

All the Exobase APIs and services are open source. In this guide, we'll walk you through those project and their code to help you get an idea of what an Exobase code base looks like in production.

!!!
We do our best to keep the code snippets in this guide up to date but we also fix bugs and implement features in our API quickly. Some snippets may not match the source exactly. 
!!!

## Exobase API

The Exobase API is the main API that wraps, secures, and manages access to our database. Our models -- like platforms, services, deployments, and users -- are all managed through this API.

#### See the source
[!ref target="blank" text="Exobase API"](https://github.com/exobase-inc/exobase-api)

### Project Structure
All endpoints are under `src/modules`. Anything shared across the api is in `src/core`. Here is a shortened visual of the directory structure.
```
src/
├─ core/
|  ├─ view/
|  ├─ model/
|  ├─ db/
|  ├─ types.ts
|  ├─ builder.ts
│  ├─ config.ts
│  ├─ magic.ts
├─ modules/
│  ├─ auth/
│  │  ├─ login.ts
│  ├─ platforms/
│  │  ├─ getById.ts
│  ├─ deployments/
│  │  ├─ getLatest.ts
```

### Authentication
We use [magic](https://magic.link/) to authenticate users. We wrote a custom hook to do the authentication through Magic.

// TODO: Use https://emgithub.com/

[!embed text="useMagicAuthentication"](https://emgithub.com/embed.js?target=https%3A%2F%2Fgithub.com%2Fexobase-inc%2Fexobase-api%2Fblob%2Fmaster%2Fsrc%2Fcore%2Fmagic.ts%23L15-52&style=github&showLineNumbers=on&showFileMeta=on&showCopy=on&fetchFromJsDelivr=on)

This hook checks that the request includes the required `didToken` in the `Authorization` header then validates it with Magic. If the `didToken` is valid, it requests the user's metadata, then calls the function chain with the user's metadata appended to the `auth` attribute of `props`.


## Exobase Webhook API


## Exobase Builder API