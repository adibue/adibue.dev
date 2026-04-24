---
title: "Use ik-auth in Cloudflare One"
description: "This guide explains how to integrate Infomaniak Auth as an Identity Provider into Cloudflare One ZTNA."
layout: default
parent: Guides
nav_order: 1
---

Infomaniak Auth (ik-auth) is an OpenID Connect (OIDC) based identity provider useful for integrating into applications for SSO.

## Cloudflare One integration

Since Cloudflare One supports generic OIDC providers, ik-auth can be used as well.
Here's how:

### Creating an ik-auth application

1. Log into the [Infomaniak Manager](https://manager.infomaniak.com/)
2. Go to _**Cloud Computing → Auth**_
3. Create a new application and select the type _**Web Front-End**_
4. Give your ik-auth app a name (e.g. _Cloudflare One_)
5. Set the Callback URL to `https://<your-team-name>.cloudflareaccess.com/cdn-cgi/access/callback`
6. Click _**Create Application**_
7. Make sure to note the **Client ID** and **Client Secret** as we will need them for Cloudflare

### Adding ik-auth to Cloudflare One

1. Access your [Cloudflare One Dashboard](https://one.dash.cloudflare.com/)
2. Go to _**Integrations → Identity providers**_
3. Add a new identity provider and use **OpenID Connect** as the method
4. Name it to your liking (e.g. "Infomaniak Auth")
5. **App ID**: Paste the ik-auth's **Client ID** here
6. **Client secret**: Paste the ik-auth's **Client Secret** here
7. **Auth URL**: Set to `https://login.infomaniak.com/authorize`
8. **Token URL**: Set to `https://login.infomaniak.com/token`
9. **Certificate URL**: Set to `https://login.infomaniak.com/oauth2/jwks`
10. Enable _**Proof Key for Code Exchange (PKCE)**_

Save the configuration and test it.
