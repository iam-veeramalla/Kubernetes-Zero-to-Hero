Problems with LDAP Authentication
==================================

Although LDAP works...

Modern companies rarely authenticate Kubernetes directly with LDAP.

Why?

Because LDAP

Uses username/password authentication
Doesn't support modern web login experiences
Doesn't provide identity tokens (JWT)
Doesn't integrate easily with cloud-native applications

Instead, enterprises use OIDC (OpenID Connect).
