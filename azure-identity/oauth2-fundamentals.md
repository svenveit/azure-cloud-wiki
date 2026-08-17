# OAuth 2.0 Fundamentals

OAuth 2.0 is an **authorization framework**, not an authentication protocol. It defines how an application can obtain limited access to a protected resource on behalf of a user, without ever seeing the user's credentials. This distinction matters: OAuth2 alone does not tell an application *who* the user is — that is what OpenID Connect adds on top (see the OIDC page).

## Core Roles

| Role | Description |
|---|---|
| Resource Owner | The user who owns the data (e.g. a mailbox, a profile) |
| Client | The application requesting access (web app, SPA, daemon, CLI) |
| Authorization Server | Issues tokens after authenticating the resource owner and obtaining consent (e.g. Microsoft Entra ID) |
| Resource Server | Hosts the protected resource and validates incoming access tokens (e.g. an API) |

## Key Concepts

- **Client ID / Client Secret** – identifies the application to the authorization server. Confidential clients (backend apps) hold a secret; public clients (SPAs, mobile apps) do not and must use PKCE instead.
- **Redirect URI** – the callback URL the authorization server sends the user back to after authentication. Must be registered exactly (no wildcards in production).
- **Scopes** – define the permissions being requested (e.g. `Mail.Read`, `User.Read`).
- **Access Token** – short-lived credential the client presents to the resource server. Treated as opaque by the client; only the resource server needs to understand it.
- **Refresh Token** – long-lived credential used to obtain a new access token without re-prompting the user.

## Grant Types

| Grant Type | Use Case | Status |
|---|---|---|
| Authorization Code (+ PKCE) | Web apps, SPAs, mobile apps — any app with a user present | Recommended |
| Client Credentials | Service-to-service / daemon apps, no user involved | Recommended |
| Device Code | Input-constrained devices (CLI tools, smart TVs, IoT) | Recommended for that scenario |
| Implicit | Legacy SPA flow, returns tokens directly in the URL fragment | Deprecated |
| Resource Owner Password Credentials (ROPC) | App collects username/password directly | Deprecated, avoid |

**PKCE (Proof Key for Code Exchange)** is mandatory in modern implementations of the Authorization Code flow, even for confidential clients. It prevents authorization code interception attacks by binding the initial request to the token exchange via a `code_verifier` / `code_challenge` pair.

## Example: Authorization Code Flow with Microsoft Entra ID

The diagram below shows a concrete implementation of the Authorization Code flow, using Microsoft Entra ID as the Authorization Server:

![OAuth 2.0 Authorization Code Flow with Microsoft Entra ID](images/oauth2-oidc-entra-flow.png)

Flow walkthrough:

1. User opens the browser and navigates to the web app.
2. The web app redirects the browser to Microsoft Entra ID (`/authorize` endpoint) with client ID, redirect URI, requested scopes, and (in production) a PKCE code challenge.
3. Entra ID prompts the user to authenticate.
4. User enters credentials.
5. User consents to the requested permissions (first time only, unless admin consent was pre-granted).
6. Entra ID redirects back to the web app's redirect URI with an authorization code, and the web app exchanges this code (server-side, with its client secret or PKCE verifier) for an access token and refresh token.
7. The web app stores the tokens.
8. On the next API call, the web app presents the access token; the resource server validates it (signature, issuer, audience, expiry).
9. The secure page is returned to the user.

This same diagram also illustrates an OpenID Connect flow, since step 6 typically returns an ID token alongside the access token — covered in detail on the OIDC page.

## Practical Notes

- Access tokens issued by Entra ID are JWTs for most Microsoft Graph / first-party scenarios, but for custom APIs registered as App Registrations they can also be opaque — don't assume JWT format without checking.
- `jwt.ms` (Microsoft's token decoder) is useful for inspecting Entra ID-issued tokens without pasting them into third-party tools.
- A common misconfiguration: mismatched redirect URI (trailing slash, http vs https, or wrong port during local dev) causes `AADSTS50011` errors — check the exact registered URI in the App Registration first.
- Client secrets have expiry dates in Entra ID (max 24 months) — an expired secret is a frequent, easy-to-miss cause of sudden auth failures in service-to-service scenarios.

## Key Takeaways

- OAuth2 handles **authorization** (what you can access), not authentication (who you are).
- Authorization Code + PKCE is the default choice for any flow involving a user.
- Client Credentials is the default for service-to-service communication.
- Implicit and ROPC grants should not be used in new implementations.
