# mi-app-ci — Flask app with Jenkins CI pipeline

A simple Flask application with a complete CI pipeline: every push to `main`
is automatically built, tested, and published to Docker Hub by Jenkins.

## Pipeline stages

1. **Build** — builds the Docker image, tagged `2.<BUILD_NUMBER>` (immutable
   tags, one per build; `latest` is never used).
2. **Test** — runs the image as a disposable container and checks the
   `/health` endpoint from inside it. If the check fails, the pipeline stops.
3. **Push** — authenticates against Docker Hub using Jenkins credentials
   (access token, never hardcoded) and publishes the image.

A `post/always` block removes the test container and logs out of the
registry, whether the build succeeds or fails.

## Quality gate — tested, not assumed

The gate was verified by deliberately breaking `/health` and pushing the
change:

- Build #1: all green → `flacuss74/mi-app:2.1` published.
- Build #2 (sabotage): Build green, **Test red, Push skipped** — the broken
  image was built locally but never reached the registry.
- Build #3 (fix): all green → `flacuss74/mi-app:2.3` published.

## Stack

- **App:** Python 3 / Flask, non-root container user, pinned dependencies.
- **CI:** Jenkins LTS running in Docker ("Docker outside of Docker": the
  Jenkins container talks to the host daemon through the mounted socket).
- **Registry:** Docker Hub — [`flacuss74/mi-app`](https://hub.docker.com/r/flacuss74/mi-app/tags)

## Run it locally

```bash
docker run -d -p 9090:8080 --name mi-app flacuss74/mi-app:2.3
curl http://localhost:9090/health   # → {"status": "ok"}
```

---
*Part of my Infrastructure → SRE transition lab. Next step: CD — automatic
deployment to Kubernetes.*
