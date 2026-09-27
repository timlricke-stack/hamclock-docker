# hamclock-docker

Expose a HamClock container on `ab0h.com` through Cloudflare when HamClock and its webserver run in the same container.

## Cloudflare tunnel config

1. Create a tunnel in Cloudflare Zero Trust and download credentials JSON.
2. Copy `/home/runner/work/hamclock-docker/hamclock-docker/cloudflared/config.yml.example` to `config.yml`.
3. Update:
   - `TUNNEL_ID`
   - `credentials-file` path
   - `service` port if your in-container webserver is not on `8080`
4. Route DNS:

```bash
cloudflared tunnel route dns TUNNEL_ID ab0h.com
```

## Run `cloudflared` in the same container namespace

If your HamClock container is named `hamclock`, run cloudflared against that container network namespace so `localhost` points to the same in-container webserver:

```bash
docker run -d --name cloudflared \
  --network container:hamclock \
  -v /etc/cloudflared:/etc/cloudflared:ro \
  cloudflare/cloudflared:latest tunnel --config /etc/cloudflared/config.yml run
```

With the included config, `https://ab0h.com` is proxied by Cloudflare to `http://127.0.0.1:8080` inside the HamClock container.
