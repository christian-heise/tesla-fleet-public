# tesla-fleet-public

Two static files on `tesla.appsbychristian.com`, served by Cloudflare Pages.

The Tesla Fleet API will only register a developer application whose **public**
signing key it can fetch over HTTPS at a fixed path on the application's own
domain. That is the only reason this host exists.

```
/.well-known/appspecific/com.tesla.3p.public-key.pem   the key Tesla fetches
/oauth/callback                                        the OAuth redirect target
```

## What is deliberately not here

- **The private key.** It never leaves an encrypted offline volume on the admin
  workstation. Only its public half is in this repository, which is why this
  repository can be public at all.
- **The logger, the dashboard, and every byte of vehicle data.** Those live on
  the home LAN and are reachable only from it, or through the household VPN.
  Nothing here proxies to them, and this host must never be made to.
- **Any logic.** The callback page renders the authorization code from the query
  string and makes no network request of any kind. The code is short-lived and
  useless without the client secret, but it is still a credential, so it is
  never transmitted, stored, or logged.

## Deployment

Cloudflare Pages, connected to this repository. No build command, no build
output directory — the repository root *is* the site. A push to `main` deploys.
`_headers` pins the key's content type, since with no build step nothing else
infers it from the extension.

Custom domain: `tesla.appsbychristian.com`. It is a separate hostname from the
portfolio site on purpose — one Pages project serves one site on every domain
attached to it, so putting this on the portfolio's project would serve the whole
portfolio here too.

## Verifying a deploy

The fingerprint must match the offline key exactly. A mismatch means the wrong
key is published, and registering it would pair the application to a private
half that does not exist:

```sh
curl -fsS https://tesla.appsbychristian.com/.well-known/appspecific/com.tesla.3p.public-key.pem \
  | openssl pkey -pubin -outform DER | openssl dgst -sha256
# expect: eebecf4b4c12bda1384a001b6169dac2204a0af05784f3e87a59b47fb639ed0e
```

If that path 404s while `/` works, Pages dropped the dot-directory from the
build output; the fallback is a Cloudflare Worker on a route for that exact
path, returning the PEM inline.

Operational context, and the runbook that consumes this key, are in the private
`tesla-logger` repository (`docs/setup/tesla.md`).
