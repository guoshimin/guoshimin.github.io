---
layout: post
title: Kubernetes 1.3 OIDC support
---
As of alpha 5, you need to manually supply the refresh token, which is not very user-friendly.

kubeconfig:

```
- name: oidc
  user:
    auth-provider:
      config:
        client-id: <client-id, required>
        client-secret: <client-secret, required>
        id-token: <id-token, if not specified, kubectl can use refresh token to get one>
        idp-issuer-url: https://accounts.google.com
        refresh-token: <refresh-token, required>
      name: oidc
```

apiserver:

```
--oidc-client-id=<client-id>
--oidc-issuer-url=https://accounts.google.com
```

To get refresh token:

1. In your browser, go to `https://accounts.google.com/o/oauth2/auth?redirect_uri=urn:ietf:wg:oauth:2.0:oob&response_type=code&client_id=<client-id>&scope=openid+email+profile&approval_prompt=force&access_type=offline`
2. Grant permission.
2. Use the code to run

       curl -XPOST -v https://www.googleapis.com/oauth2/v3/token \
         --data-urlencode code=<code> \
         --data-urlencode redirect_uri=urn:ietf:wg:oauth:2.0:oob \
         --data-urlencode client_id=<client-id> \
         --data-urlencode client_secret=<client-secret> \
         --data-urlencode grant_type=authorization_code
