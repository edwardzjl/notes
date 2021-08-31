# My first note

## x-forward

- set `X-Forwarded-Prefix`
- set `server.forward-headers-strategy=FRAMEWORK`
- `request.getContextPath()` (application context path) will become what you set in `X-Forwarded-Prefix`
- see [this answer](https://stackoverflow.com/a/59126519/6564721) for details
