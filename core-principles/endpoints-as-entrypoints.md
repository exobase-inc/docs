---
label: Endpoints As Entrypoints
layout: default
order: 2
---

# Endpoints As Entrypoints

Every endpoint should be a self contained application that can be executed independent of all other endpoints. This is a bit foreign (unless you come from a serverless function background). Most popular frameworks like Express, Flask, Django, Laravel, Vapor, and Gin are the opposite of this. In these frameworks, you define an application and attach routes to it. All requests begin in the application. With Exobase, we leverage our framework agnostic, isolated behavior, and composed endpoints to package an entire application worth of behavior into each endpoint.