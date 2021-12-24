---
label: Getting Started
layout: default
order: 999
description: This is a custom description for this page
icon: rocket
---

# Getting Started
Welcome to Exobase. There is a lot to learn but the upside is huge.

## What is Exobase?
Exobase is two parts: multi-cloud deployment & the exobase library.

### Muti-Cloud Deployment
We get to know the unique APIs that each cloud provider/service exposes and we implement a bridge for each one so you get the same interface for every cloud provider/service. This is the underlying magic that lets you move a service from cloud to cloud with Exobase.

### The Exobase Library
If your a more traditional soul'd developer, less apt to accept change, you may have a hard time with this. In order to make multi-cloud deployment possible we had to rethink a lot of the _standards_ and _best practices_ around APIs and web services. We chose to think of any online service as a program made up of modules and functions (like any program). Done. We defined a simple request/response interface and threw out everything else.

!!! A Little Vocabulary
Here are a few words that are used often in our industry so before you dive into the docs here is what we mean when we use them.
!!!
###### Cloud Provider
Cloud Provider refers to companies like AWS, GCP, IBM, Azure. This also includes second tier providers that run their infrastructure on the previous listed providers like Vercel, Netlify, or Heroku.

###### Cloud Service
Cloud Service referes to a specific service that a cloud provider offers. Examples would be AWS Lambda, GCP Cloud Run, and AWS Elastic Container Service.

###### Cloud Provider/Service 
Cloud Provider/Service refers to any combination of the above two terms. This accounts for the near hundreds of services offered by dozens of online cloud providers.

###### Platform 
Platform referes to your platform in Exobase. It is made up of many services that can all be hosted and run on any cloud provider/service.

## Writing Exobase Code
The first thing might like to do is see the code. You don't want to use Exobase if the code you have to write is so ugly it scares children (_it doesn't, it is quiet nice_). Start here:

[!ref](./core-principles/index.md)
[!ref](./exobase-code/index.md)
- See our template api repository
- See the code for the actual Exobase API _(it's open source)_

## How Does Exobase Work?
Understanding the system completely will help you know the abilities and limits and hopefully build your platform faster. Start here:

- Intro to FRAFI (framework agnostic function invocation)
- How Exobase Deploys
- See our Pulumi template repository

## Quick Deploy
If you're the 'learn by doing' type, start here:

- Deploy an API with Exobase in 2 minutes
- Deploy a React app with Exobase in 2 minutes

## Exobase Guides
We've written guides to some of the most common use cases of web APIs and web services. If you want to dive deep, start here:

- Build a TODO app with Exobase
- Build a GitHub integration service with Exobase
- Build a Webhook Handling Service with Exobase
- Build an Exobase Hook
- Build an Exobase Root Hook
- Build support for a new cloud provider/service