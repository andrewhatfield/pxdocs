---
title: Authorization with pxctl
linkTitle: Authorization
keywords: portworx, container, Kubernetes, storage, auth, authz, authorization, authentication, login, token, oidc,context, generate, self-signed, jwt, shared-secret, c
description: Learn to enable auth in your px cluster
weight: 3
---

## Overview 

This document outlines how to interact with an auth-enabled PX cluster. 

## Context

pxctl allows you to store contexts and associated clusters, privileges, and tokens local to your users

This enables you to easily switch between these configurations with a few commands:

```text
/opt/pwx/bin/pxctl context --help
```
```
Portworx pxctl context commands for setting authentication and connection info

Usage:
  pxctl context [flags]
  pxctl context [command]

Available Commands:
  create      create a context
  delete      delete a context
  list        list all contexts
  set         set the current context
  unset       unset the current context

```

### Context management
You can easily create and delete contexts with the below commands:

__Creating or updating a context:__
```text
/opt/pwx/bin/pxctl context create <context> --token <token> --endpoint <endpoint>
```
    
__Deleting a context:__
```text
/opt/pwx/bin/pxctl context delete <context>
```

__Listing your contexts:__

Your context is store in `~/.pxctl/contextconfig`, but you can easily view it with the `list` subcommand:

```text
/opt/pwx/bin/pxctl context list
```
```
contextconfig:
  current: user
  configurations:
  - context: user
    token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImpzdGV2ZW5zQHBvcnR3b3J4LmNvbSIsImV4cCI6MTU1MzcyNTMyMSwiZ3JvdXBzIjpbInB4LWVuZ2luZWVyaW5nIiwia3ViZXJuZXRlcy1jc2kiXSwiaWF0IjoxNTUzNjM4OTIxLCJpc3MiOiJwb3J0d29yeC5jb20iLCJuYW1lIjoiSmltIFN0ZXZlbnMiLCJyb2xlcyI6WyJzeXN0ZW0udXNlciJdLCJzdWIiOiJqc3RldmVuc0Bwb3J0d29yeC5jb20vanN0ZXZlbnMifQ.pZDbCIL7ldcImvIaNSjk18Ah3LqxX63MV378NiauRwk
    identity:
      subject: jstevens@portworx.com/jstevens
      name: Jim Stevens
      email: jstevens@portworx.com
    endpoint: http://localhost:9001
```


### Current context

Now that you've created your contexts, you can easily switch between them with the commands below.

__Setting current context:__

```text
/opt/pwx/bin/pxctl context set <context>
```

__Unsetting current context:__

```text
/opt/pwx/bin/pxctl context unset
```

## Generating tokens
PX supports two methods of authorization: OIDC and self-signed. 

* For generating a token through your OIDC provider, see your provider's documentation on generating bearer tokens:
  * [Keycloak](https://www.keycloak.org/docs/1.9/server_development_guide/topics/admin-rest-api.html)
  * [Auth0](https://auth0.com/docs/api/authentication#get-token)
  * [Okta](https://developer.okta.com/docs/api/getting_started/getting_a_token/#token-expiration)
* For self-signed, pxctl has a command for generating tokens

__Generating self-signed tokens:__ pxctl allows you to generate self-signed tokens in a few different ways: ECDSA, RSA, and Shared-Secret. In addition to these parameters, you must pass an issuer and authconfig.yaml. See below for an example with configuration `authconfig.yaml`.

```text
pxctl auth token generate --auth-config=<authconfig.yaml> --issuer <issuer> \
    --ecdsa-private-keyfile <ecdsa key file> OR \
    --rsa-private-keyfile <rsa key file> OR \
    --shared-secret <secret>
```

__Sample configuration (authconfig.yaml):__
```text
name: Jim Stevens
email: jstevens@portworx.com
sub: jstevens@portworx.com/jstevens
roles: ["system.user"]
groups: ["*"]
```

## Debugging token issues:

You may gotten an unexpected `"Permission denied"` or other auth-related error. To take a look into your token permissions, you can always decode it with a JWT token decoding tool such as [jwt.io](https://jwt.io/)

__Note:__ Be careful where you paste your token. [jwt.io](https://jwt.io/) does client-side validation and debugging and does not store your token anywhere.