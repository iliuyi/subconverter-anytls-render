# subconverter-anytls-render

Render Docker deployment for subconverter with Surge AnyTLS parsing support.

## What changed

This build adds Surge `[Proxy]` parsing for `anytls` nodes:

```text
Hong Kong 01 = anytls, example.com, 15026, password=xxx, tfo=true, sni=example.com, server-cert-fingerprint-sha256=<sha256>
```

Supported parsed fields:

- `password`
- `sni`
- `server-cert-fingerprint-sha256`
- `fingerprint`
- `skip-cert-verify`
- `tls13`
- `udp-relay`
- `tfo`

## Render deployment

1. Create a new Render Web Service.
2. Choose "Build and deploy from a Git repository".
3. Select this GitHub repository.
4. Render detects the root `Dockerfile`.
5. Keep the environment variable `PORT=25500`, or use the included `render.yaml` blueprint.
6. Deploy.

After deployment, check:

```bash
curl https://YOUR-SERVICE.onrender.com/version
```

## Conversion test

```bash
curl -G \
  --data-urlencode target=clash \
  --data-urlencode url='https://gist.githubusercontent.com/metricss/c5e1a79c85310da1debf2db2eb2223c8/raw/9e84d58b1f56ccee1c95ff7c80295ac39dbc86e6/test.conf' \
  https://YOUR-SERVICE.onrender.com/sub
```

Expected output includes five `type: anytls` proxies.

## Docker build args

- `UPDATE_RULES`: default `0`. Skips build-time online rule updates to avoid slow or hanging GitHub rule downloads. Set to `1` to run `scripts/update_rules.py` during build.
- `APK_MIRROR`: default `https://mirrors.aliyun.com/alpine`. Alpine package mirror.
- `THREADS`: default `4`. Build parallelism.
- `SHA`: default `render`. Version suffix.
