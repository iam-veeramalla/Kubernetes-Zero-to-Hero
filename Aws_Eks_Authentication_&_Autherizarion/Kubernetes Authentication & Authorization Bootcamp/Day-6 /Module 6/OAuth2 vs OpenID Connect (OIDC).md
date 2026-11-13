OAuth2 vs OpenID Connect (OIDC)
===============================

Many engineers confuse these.

OAuth2
--------
Purpose:

Authorization

It answers:

"Can this application access my resources?"

Example:

Login with Google

↓

Application requests permission

↓

Access Gmail

Access Drive

Access Calendar

OAuth2 grants access to resources.

OpenID Connect (OIDC)
------------------------
OIDC is built on top of OAuth2.

Purpose:

Authentication

It answers:

"Who is this user?"

OIDC returns an ID Token (JWT) containing user identity.

Example JWT claims:

{
  "name": "Rahul",
  "email": "rahul@company.com",
  "groups": [
    "DevOps",
    "Platform"
  ]
}

Kubernetes reads these claims to authenticate the user.
