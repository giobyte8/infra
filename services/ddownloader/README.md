# Ddownloader

## Permissions on runtime volume

The application usually will run as a non-root user.
From the `compose.yaml` file:
```yaml
services:
  ddownloader:
    user: "${UID}:${GID}"
    # ...
```

However the volumes are created by docker compose as root, so the application
won't be able to write to the volume.

**Solution:**
A simple solution is to start the container once as root and set the
right permissions to directories inside the volume

```bash
# Make sure to use the right UID and GID for your user
docker compose run --rm \
  --user root \
  --entrypoint /bin/sh \
    ddownloader \
    -c 'chown -R 1000:1000 /opt/ddownloader/config/runtime'

# After that, initiate the service as usual
docker compose up -d ddownloader
```
