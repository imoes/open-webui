# Keycloak SSO

This project already supports OpenID Connect through the built-in `oidc` provider. Keycloak can therefore be integrated without custom backend code as long as the provider, redirect URI, and claim mapping are configured correctly.

## What this enables

- Single sign-on with Keycloak via the existing `/oauth/oidc/login` flow
- Automatic user creation on first login
- Optional role mapping from Keycloak roles to Open WebUI `user` and `admin`
- Optional group synchronization from Keycloak groups to Open WebUI groups
- Optional OIDC back-channel logout

## 1. Create the Keycloak client

In your Keycloak realm:

1. Create a new client.
2. Set the client protocol to `openid-connect`.
3. Use a confidential client if you want to authenticate with a client secret.
4. Add the Open WebUI callback URL as a valid redirect URI:
   `https://<open-webui-host>/oauth/oidc/callback`
5. Add the Open WebUI origin as a valid web origin.
6. If you want Open WebUI logout to trigger Keycloak logout, ensure the realm exposes an `end_session_endpoint`.

If you prefer PKCE without a client secret, Open WebUI supports `S256` through `OAUTH_CODE_CHALLENGE_METHOD=S256`.

## 2. Configure Open WebUI

Set the following environment variables for a standard Keycloak setup:

```env
ENABLE_OAUTH_SIGNUP=true
OAUTH_PROVIDER_NAME=Keycloak

OAUTH_CLIENT_ID=<keycloak-client-id>
OAUTH_CLIENT_SECRET=<keycloak-client-secret>

OPENID_PROVIDER_URL=https://<keycloak-host>/realms/<realm-name>/.well-known/openid-configuration
OPENID_REDIRECT_URI=https://<open-webui-host>/oauth/oidc/callback

OAUTH_SCOPES=openid email profile
OAUTH_TOKEN_ENDPOINT_AUTH_METHOD=client_secret_post

OAUTH_USERNAME_CLAIM=preferred_username
OAUTH_EMAIL_CLAIM=email
OAUTH_PICTURE_CLAIM=picture
```

For your environment this typically looks like:

```env
OPENID_PROVIDER_URL=https://keycloak.ippen.media/realms/<realm-name>/.well-known/openid-configuration
```

Replace `<realm-name>`, `<keycloak-client-id>`, `<keycloak-client-secret>`, and `<open-webui-host>` with your actual values.

## 3. Role mapping

Open WebUI can map Keycloak roles into its own `user` and `admin` roles.

Keycloak realm roles are commonly available under `realm_access.roles`, which Open WebUI supports through dot-notation claim lookup.

Example:

```env
ENABLE_OAUTH_ROLE_MANAGEMENT=true
OAUTH_ROLES_CLAIM=realm_access.roles
OAUTH_ALLOWED_ROLES=user,admin,openwebui-user
OAUTH_ADMIN_ROLES=admin,openwebui-admin
```

Behavior:

- If role management is enabled and the user has an allowed role, they become an Open WebUI `user`.
- If they have an admin role, they become an Open WebUI `admin`.
- If role management is enabled and no configured role matches, login is denied.

If your roles are emitted in a different claim, point `OAUTH_ROLES_CLAIM` at that claim instead.

## 4. Group mapping

If Keycloak includes groups in the token, Open WebUI can synchronize them.

Example:

```env
ENABLE_OAUTH_GROUP_MANAGEMENT=true
OAUTH_GROUPS_CLAIM=groups
ENABLE_OAUTH_GROUP_CREATION=true
```

Notes:

- `groups` is a common claim name when a Keycloak group mapper is configured.
- Group sync only works if the groups are present in the ID token or userinfo response.
- If automatic creation is enabled, missing Open WebUI groups are created on login.

## 5. Keycloak mappers

Depending on your realm configuration, you may need to add protocol mappers so the required claims are present in the token.

Common mappers:

- `email`
- `preferred_username`
- `groups`
- realm roles in `realm_access.roles`

For group sync, ensure the mapper adds groups into the token. For role-based access, ensure the roles you expect are actually present in the token returned to Open WebUI.

## 6. Logout

Open WebUI supports OpenID Connect logout through provider discovery. If your discovery document exposes an `end_session_endpoint`, standard logout works automatically.

If your environment requires an explicit override, set:

```env
OPENID_END_SESSION_ENDPOINT=https://<keycloak-host>/realms/<realm-name>/protocol/openid-connect/logout
```

## 7. Optional back-channel logout

Open WebUI also supports OIDC back-channel logout:

```env
ENABLE_OAUTH_BACKCHANNEL_LOGOUT=true
```

When enabled, Keycloak can be configured to call:

```text
https://<open-webui-host>/oauth/backchannel-logout
```

Important:

- This is most useful in multi-session or centrally managed environments.
- JWT revocation for back-channel logout relies on Redis-backed revocation support.

## 8. Reverse proxy considerations

If Open WebUI runs behind a reverse proxy:

- Make sure the public URL used by the browser matches `OPENID_REDIRECT_URI`.
- Ensure `X-Forwarded-*` headers are forwarded correctly.
- Use HTTPS in production.

Redirect mismatches are the most common cause of failed logins.

## 9. Troubleshooting

### Login button does not appear

Check that all required OIDC values are set:

- `OAUTH_CLIENT_ID`
- `OPENID_PROVIDER_URL`
- `OAUTH_CLIENT_SECRET` or `OAUTH_CODE_CHALLENGE_METHOD`

### Login succeeds in Keycloak but Open WebUI rejects the user

Check:

- `ENABLE_OAUTH_SIGNUP`
- `OAUTH_ALLOWED_DOMAINS`
- `ENABLE_OAUTH_ROLE_MANAGEMENT`
- `OAUTH_ROLES_CLAIM`
- the actual roles present in the token

### Groups or roles are missing

This is usually a Keycloak mapper issue. Verify the claims are present in the ID token or userinfo response and that the configured claim names match those token fields.

### Logout does not return to Keycloak

Check:

- the discovery document includes `end_session_endpoint`
- `OPENID_END_SESSION_ENDPOINT` is set if your provider needs an override

## 10. Recommended production baseline

```env
ENABLE_OAUTH_SIGNUP=true
OAUTH_MERGE_ACCOUNTS_BY_EMAIL=false
OAUTH_PROVIDER_NAME=Keycloak

OAUTH_CLIENT_ID=<client-id>
OAUTH_CLIENT_SECRET=<client-secret>
OPENID_PROVIDER_URL=https://<keycloak-host>/realms/<realm>/.well-known/openid-configuration
OPENID_REDIRECT_URI=https://<open-webui-host>/oauth/oidc/callback
OAUTH_TOKEN_ENDPOINT_AUTH_METHOD=client_secret_post

OAUTH_USERNAME_CLAIM=preferred_username
OAUTH_EMAIL_CLAIM=email
OAUTH_PICTURE_CLAIM=picture

ENABLE_OAUTH_ROLE_MANAGEMENT=true
OAUTH_ROLES_CLAIM=realm_access.roles
OAUTH_ALLOWED_ROLES=user,admin
OAUTH_ADMIN_ROLES=admin
```

If you also use Keycloak groups:

```env
ENABLE_OAUTH_GROUP_MANAGEMENT=true
OAUTH_GROUPS_CLAIM=groups
ENABLE_OAUTH_GROUP_CREATION=true
```
