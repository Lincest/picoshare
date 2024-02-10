## Run PicoShare

### DEV: From source

```bash
PS_SHARED_SECRET=somesecretpass PORT=4001 \
  go run cmd/picoshare/main.go
```

### PROD: Using Docker Compose

To run PicoShare under docker-compose, copy the following to a file called `docker-compose.yml` and then run `docker-compose up`.

```yaml
version: "3.2"
services:
  picoshare:
    image: ghcr.io/lincest/picoshare:latest
    environment:
      - PORT=4001
      - PS_SHARED_SECRET=dummypass # Change to any password
    ports:
      - 4001:4001
    command: -db /data/store.db
    volumes:
      - ./data:/data
```

## Parameters

### Command-line flags

| Flag  | Meaning                 | Default Value     |
| ----- | ----------------------- | ----------------- |
| `-db` | Path to SQLite database | `"data/store.db"` |

### Environment variables

| Environment Variable | Meaning                                                                              |
| -------------------- | ------------------------------------------------------------------------------------ |
| `PORT`               | TCP port on which to listen for HTTP connections (defaults to 4001).                 |
| `PS_BEHIND_PROXY`    | Set to `"true"` for better logging when PicoShare is running behind a reverse proxy. |
| `PS_SHARED_SECRET`   | (required) Specifies a passphrase for the admin user to log in to PicoShare.         |


### Docker build args

If you rebuild the Docker image from source, you can adjust the build behavior with `docker build --build-arg`:

| Build Arg            | Meaning                                                                     | Default Value |
| -------------------- | --------------------------------------------------------------------------- | ------------- |
| `litestream_version` | Version of [Litestream](https://litestream.io/) to use for data replication | `0.3.9`       |

## PicoShare's scope and future

PicoShare is maintained by Michael Lynch as a hobby project.

Due to time limitations, I keep PicoShare's scope limited to only the features that fit into my workflows. That unfortunately means that I sometimes reject proposals or contributions for perfectly good features. It's nothing against those features, but I only have bandwidth to maintain features that I use.

## Deployment

PicoShare is easy to deploy to cloud hosting platforms:

- [fly.io](docs/deployment/fly.io.md)

## Tips and tricks

### Reclaiming reserved database space

Some users find it surprising that when they delete files from PicoShare, they don't gain back free space on their filesystem.

When you delete files, PicoShare reserves the space for future uploads. If you'd like to reduce PicoShare's usage of your filesystem, you can manually force PicoShare to give up the space by performing the following steps:

1. Shut down PicoShare.
1. Run `sqlite3 data/store.db 'VACUUM'` where `data/store.db` is the path to your PicoShare database.

You should find that the `data/store.db` should shrink in file size, as it relinquishes the space dedicated to previously deleted files. If you start PicoShare again, the System Information screen will show the smaller size of PicoShare files.
