# Docker & Containers for Developers — Lab Guide

**NAYA College · 3-Day Practical Course · Docker 24+**

This is the hands-on companion to the slides. Work through it in order; each lab builds on the last. By the end you will have taken **one application** all the way from a single container to a CI pipeline that builds, tests, scans, and ships it.

---

## How to use this guide

- **`$`** marks a command you type in your terminal. **`/ #`** marks a command typed *inside* a container shell.
- Each lab has a **Stop & think** box. Try to answer it *before* moving on — that is where the real learning happens.
- **Pitfalls** call out the mistakes we see every cohort make. Read them; they will save you time.
- Solutions live on the `solutions` branch of the course repo. Use them only after you have tried.
- Everywhere you see the **course app**, it is whatever stack we agreed on for your cohort (the commands below assume a small web service + a database; swap the base image and start command for your language).

---

## Pre-flight checklist (do this *before* Day 1)

You should be able to run all of these without errors:

```bash
$ docker version          # shows BOTH a Client and a Server section
$ docker info             # engine status, storage driver, OS/Arch
$ docker run hello-world  # prints a welcome message
$ docker compose version  # note: a SPACE, not a hyphen — must be v2.x+
```

Also: create a **Docker Hub** account (or have your private registry credentials ready) and clone the course repo. If any command above fails, flag it at the start of Day 1 so we fix it before the first lab.

> **Instructor note — two images to publish once:** Labs 4 and 5 run pre-built images. Build and push them before the course from the `lab-images/` bundle: `levep79/broken-demo` (Lab 4) and `levep79/course-app` (Labs 5, and the same source students build in Labs 6–9). See `lab-images/README.md`. Everything else is an official image or built by students.

---

# Day 1 — Foundations & the CLI

## Demo 1 (instructor) — Your first container
The instructor runs `docker run alpine echo "hello"` and `docker run -it alpine sh`, narrating what the client sends to the daemon, what gets pulled, and what "interactive" means. Watch; you will do this yourself in Lab 2.

---

## Lab 1 — Install Check
**~10 min · Goal:** Confirm the Engine, CLI, and Compose v2 all work before we build anything.

1. Run `docker version`. Read the two halves — **Client** and **Server**.
2. Run `docker info`. Note the **Storage Driver** and **OS/Arch**.
3. Run `docker run hello-world`.
4. Run `docker compose version` — confirm it reports **v2.x** (space, not hyphen).
5. Run `docker login` and sign in.

**Expected result:** Every command succeeds; you are logged in.

> **Stop & think:** `docker version` shows two sections — client and server. Why two? When you type a `docker` command, what is actually talking to what?

**Pitfall:** If `docker` works but `docker compose` does not, you may have the retired v1 binary (`docker-compose`, hyphen) and no v2 plugin. We want **`docker compose`**.

---

## Lab 2 — Explore a Container
**~15 min · Goal:** Get comfortable moving around inside a running Linux container.

1. Start a detached Alpine container:
   ```bash
   $ docker run -dit --name box alpine sh
   ```
2. Exec into it:
   ```bash
   $ docker exec -it box sh
   ```
3. Inside, find your bearings:
   ```bash
   / # cat /etc/os-release     # which distro?
   / # whoami                  # which user?
   / # ps aux                  # what is running?
   ```
4. Create a file and change its permissions:
   ```bash
   / # echo hi > /tmp/note.txt
   / # chmod 600 /tmp/note.txt
   / # ls -l /tmp/note.txt
   ```
5. Detach **without** killing the container: press `Ctrl-P` then `Ctrl-Q`. Confirm with `docker ps`.

**Expected result:** `box` is still running after you detach.

> **Stop & think:** Inside the container you are `root`. List the running processes — how many are there, and what is **PID 1**? How is that different from your laptop?

**Pitfall:** `exit` from a shell you started with `exec` is fine (it only ends that shell). `exit` from the container's **main** process ends the container.

**Cleanup:** `docker rm -f box`

---

## Lab 3 — Pull & Pin
**~35 min · Goal:** Practise consuming images you didn't build — pull, inspect, pin by digest, clean up.

1. Pull two specific tags and list them:
   ```bash
   $ docker pull nginx:1.27
   $ docker pull alpine:3.20
   $ docker images
   ```
2. Find the immutable digest of the nginx image:
   ```bash
   $ docker inspect -f '{{index .RepoDigests 0}}' nginx:1.27
   # e.g. nginx@sha256:abc123...
   ```
3. Pull that **exact** image by digest:
   ```bash
   $ docker pull nginx@sha256:<paste-the-digest>
   ```
4. Remove an image, then prune dangling ones:
   ```bash
   $ docker rmi alpine:3.20
   $ docker image prune
   ```
5. On Docker Hub, find the nginx page and note whether it is **Official**, and
   what the difference is between the `latest`, `1.27`, and `1.27-alpine` tags.

**Expected result:** Both tags pulled, the digest identified and re-pulled, and a
clean image list.

> **Stop & think:** You pinned an image by its digest instead of its tag. Six
> months from now, why might `nginx:1.27` give you a *different* image than the
> digest you recorded today?

**Pitfall:** Anonymous Docker Hub pulls are rate-limited. If pulls start failing
with a "toomanyrequests" message, run `docker login` first.

---

## Lab 4 — Container Lifecycle
**~20 min · Goal:** Drive a container through its whole life from the CLI.

1. Run nginx detached on port 8080:
   ```bash
   $ docker run -d --name web -p 8080:80 nginx
   ```
2. Confirm it is running and open it:
   ```bash
   $ docker ps
   $ curl localhost:8080        # or open http://localhost:8080
   ```
3. Stop it, then list with and without `-a`:
   ```bash
   $ docker stop web
   $ docker ps          # not shown
   $ docker ps -a       # shown, status Exited
   ```
4. Start it again, then restart it:
   ```bash
   $ docker start web
   $ docker restart web
   ```
5. Remove it and prove it is gone:
   ```bash
   $ docker rm -f web
   $ docker ps -a | grep web    # no output
   ```

**Expected result:** `web` no longer appears in `docker ps -a`.

> **Stop & think:** After `docker stop` then `docker start`, is it the **same** instance? After `docker rm` then `docker run`, is it? What would persist across each, and what would not?

**Pitfall:** "Port is already allocated" → another process owns 8080. Pick another host port: `-p 8081:80`.

---

## Lab 5 — Debug a Broken Container
**~20 min · Goal:** Diagnose a container that starts and immediately exits — using only logs and inspect.

1. Run the broken image (it will exit):
   ```bash
   $ docker run --name broken levep79/broken-demo
   ```
2. Check how it died:
   ```bash
   $ docker ps -a            # note the Exit Code
   ```
3. Read the logs:
   ```bash
   $ docker logs broken
   ```
4. Inspect what it tried to run:
   ```bash
   $ docker inspect broken --format '{{json .Config.Cmd}}'
   ```
5. Form a hypothesis, fix the run command the logs point to, and verify it stays up:
   ```bash
   $ docker rm broken
   $ docker run -d --name broken -e DB_HOST=db levep79/broken-demo
   $ docker ps          # now it stays running
   ```

**Expected result:** A corrected `docker run` (supplying `DB_HOST`) keeps the container running.

> **Stop & think:** Before reading the logs, what were your top two guesses? Did the **exit code** narrow it down before the logs confirmed it?

**Pitfall:** A container that exits `0` did its job and stopped — that is not always a bug. Non-zero means failure. Read the code first.

---

## Lab 6 — Publish & Configure
**~50 min · Goal:** Run the course web app, expose it, and change its behaviour through environment variables.

1. Run the course app detached and publish its port:
   ```bash
   $ docker run -d --name app -p 8080:3000 levep79/course-app
   ```
2. Reach it from your browser at `http://localhost:8080`.
3. Pass configuration with `-e` and observe the change:
   ```bash
   $ docker rm -f app
   $ docker run -d --name app -p 8080:3000 -e APP_ENV=staging -e LOG_LEVEL=debug levep79/course-app
   $ docker logs app
   ```
4. Move the variables into a file and use `--env-file`:
   ```bash
   # app.env
   APP_ENV=staging
   LOG_LEVEL=debug
   ```
   ```bash
   $ docker rm -f app
   $ docker run -d --name app -p 8080:3000 --env-file ./app.env levep79/course-app
   ```
5. Confirm the published port and the environment:
   ```bash
   $ docker port app
   $ docker inspect app --format '{{json .Config.Env}}'
   ```

**Expected result:** The app's logged behaviour changes with the env vars; the port is mapped.

> **Stop & think:** You set the same variable two ways (`-e` and an env-file). If both set `APP_ENV`, which wins? How could you find out for certain rather than guessing?

**Pitfall:** "It loads but the browser can't reach it" → the app inside the container is bound to `127.0.0.1`. It must listen on `0.0.0.0` to be reachable through a published port.

---

## Lab 7 — Tame a Container
**~30 min · Goal:** Run a service that survives crashes, respects a memory cap, and lets you move files in and out.

1. Run a container with a restart policy:
   ```bash
   $ docker run -d --name web --restart=unless-stopped -p 8080:80 nginx
   ```
2. Add resource limits (re-create it to apply them) and watch usage:
   ```bash
   $ docker rm -f web
   $ docker run -d --name web --restart=unless-stopped \
       --memory=256m --cpus=0.5 -p 8080:80 nginx
   $ docker stats web        # Ctrl-C to exit
   ```
3. Kill the main process and watch the policy restart it:
   ```bash
   $ docker kill web
   $ docker ps               # it comes back on its own
   ```
4. Copy a file in, then back out:
   ```bash
   $ echo "<h1>Tamed</h1>" > index.html
   $ docker cp ./index.html web:/usr/share/nginx/html/index.html
   $ curl localhost:8080
   $ docker cp web:/etc/nginx/nginx.conf ./nginx.conf
   ```
5. Raise the memory limit live:
   ```bash
   $ docker update --memory=512m web
   ```

**Expected result:** The container restarts after a kill, enforces its limits, and
serves your copied-in file.

> **Stop & think:** You set a memory cap of 256m. If the app exceeds it, what does
> Docker do — and how would `docker ps` / `docker logs` show you it happened?

**Pitfall:** `--restart=always` will resurrect a container even after you think
you've stopped it via the daemon restarting. Use `unless-stopped` so a deliberate
`docker stop` actually sticks.

---

## Lab 8 — Investigate a Running Container
**~30 min · Goal:** Use `inspect`, `logs`, `events`, and `diff` to understand a container instead of guessing.

1. Start a container and change something inside it:
   ```bash
   $ docker run -d --name web -p 8080:80 nginx
   $ docker exec web sh -c 'echo "hi" > /tmp/note.txt'
   ```
2. Pull specific facts with `--format` instead of reading all the JSON:
   ```bash
   $ docker inspect -f '{{.State.Status}}' web
   $ docker inspect -f '{{.NetworkSettings.IPAddress}}' web
   $ docker inspect -f '{{json .Config.Env}}' web
   ```
3. Read just the recent logs:
   ```bash
   $ docker logs --since 5m -t web
   ```
4. In a **second terminal**, watch daemon events while you stop the container:
   ```bash
   $ docker events            # leave running
   # (first terminal) $ docker stop web
   ```
5. See what changed in the writable layer:
   ```bash
   $ docker start web
   $ docker exec web sh -c 'echo more >> /tmp/note.txt'
   $ docker diff web          # A /tmp/note.txt
   ```

**Expected result:** You can state the container's status, IP, env, recent log
lines, and exactly which files it changed — all without guessing.

> **Stop & think:** `docker diff` lists the files the container changed since it
> started. Where do those changes physically live, and what happens to them when
> the container is removed?

**Pitfall:** `docker inspect` with no `--format` dumps a screen of JSON. Reach for
`-f '{{...}}'` to get the one field you actually want.

---

## Lab 9 — Reclaim Disk
**~15 min · Goal:** Find what Docker is storing and clean it up without losing what you need.

1. See current usage:
   ```bash
   $ docker system df
   ```
2. Generate some cruft:
   ```bash
   $ docker run --name t1 alpine echo hi     # exits, leaves a stopped container
   $ docker run --name t2 alpine echo hi
   $ docker pull nginx:1.25                   # an older image you won't use
   ```
3. Look again and watch the numbers grow:
   ```bash
   $ docker system df
   ```
4. Clean up in stages and read each confirmation:
   ```bash
   $ docker container prune     # removes t1, t2
   $ docker image prune         # removes dangling images
   ```
5. Compare:
   ```bash
   $ docker system df
   ```

**Expected result:** Stopped containers and dangling images are gone; `system df`
reflects the reclaimed space.

> **Stop & think:** `docker system prune -a` also removes images **not currently
> used by a container** — including ones you pulled deliberately. Why can that be a
> nasty surprise right before a flight with no wifi?

**Pitfall:** `docker system prune -a --volumes` will delete named volumes that
aren't attached to a running container — i.e. your database data. Read the prompt
before typing `y`.

---

## Lab 10 — Capstone: Operate a Service (CLI only)
**~45 min · Goal:** Put the whole day together. Run, configure, harden, observe, and clean up one real service — no Dockerfile, no Compose.

1. Run the course app, detached, named, with a published port:
   ```bash
   $ docker run -d --name app -p 8080:3000 levep79/course-app
   $ curl localhost:8080
   ```
2. Re-run it configured from a file, with a restart policy:
   ```bash
   $ docker rm -f app
   $ printf "APP_ENV=staging\nLOG_LEVEL=debug\n" > app.env
   $ docker run -d --name app --restart=unless-stopped \
       --env-file app.env -p 8080:3000 levep79/course-app
   ```
3. Add resource limits and confirm them:
   ```bash
   $ docker rm -f app
   $ docker run -d --name app --restart=unless-stopped --env-file app.env \
       --memory=256m --cpus=0.5 -p 8080:3000 levep79/course-app
   $ docker stats app        # Ctrl-C to exit
   ```
4. Observe and interact:
   ```bash
   $ docker logs --tail 20 app
   $ docker exec -it app sh
   / # exit
   $ docker cp app:/app/server.js ./server-from-container.js
   ```
5. Inspect, then tear everything down:
   ```bash
   $ docker inspect -f '{{.State.Status}} {{.HostConfig.RestartPolicy.Name}}' app
   $ docker port app
   $ docker rm -f app
   ```

**Expected result:** You ran, configured, limited, observed, and cleaned up a real
service entirely from the CLI.

> **Stop & think:** You operated this service end-to-end with the CLI alone. Which
> single step felt most repetitive — and what would make it reproducible tomorrow?
> (Hold that thought: Day 3's Compose answers exactly this.)

**Pitfall:** Each time you change `-e`, `--memory`, or ports you had to `rm -f` and
re-run the whole command. That friction is the motivation for Compose — notice it
now so Day 3 lands.

---

# Day 2 — Images, Dockerfiles, Data & Networking

## Demo 2 (instructor) — Image layers
The instructor pulls `node:20-alpine`, runs `docker history` to show per-layer sizes, then pulls a second image sharing the base to show **reused layers**. Watch which layers download and which are already cached.

> **Stop & think:** When the second image was pulled, did the shared base layer download again? How does the output tell you?

---

## Lab 11 — Build Your First Image
**~60 min · Goal:** Write a Dockerfile for the course app and build it into a runnable image.

1. In the app folder, create a `Dockerfile`:
   ```dockerfile
   FROM node:20-alpine
   WORKDIR /app
   COPY package*.json ./
   RUN npm ci
   COPY . .
   EXPOSE 3000
   CMD ["node", "server.js"]
   ```
2. Build it with a name and tag:
   ```bash
   $ docker build -t myapp:0.1 .
   ```
3. Run it and reach it in the browser:
   ```bash
   $ docker run -d -p 3000:3000 myapp:0.1
   ```
4. Change one line of **source** code and rebuild:
   ```bash
   $ docker build -t myapp:0.2 .
   ```
5. Read the build output: note which layers are `CACHED` and which rebuilt.

**Expected result:** Two image tags; the second build reuses the dependency layers.

> **Stop & think:** You changed one line of app code. Which layers rebuilt? If you had put `COPY . .` **before** `RUN npm ci`, how would the caching change?

**Pitfall:** Copying everything before installing deps means **every** code change reinstalls all dependencies. Manifest first, source second.

---

## Lab 7 — Refine the Image
**~35 min · Goal:** Tighten the image with `.dockerignore`, `ENTRYPOINT`, and cleaner layers.

1. Add a `.dockerignore`:
   ```
   node_modules
   .git
   *.log
   .env
   dist
   ```
2. Rebuild and compare the **build context** size reported at the top of the build.
3. Switch the start definition to `ENTRYPOINT` + `CMD` and test how arguments behave:
   ```dockerfile
   ENTRYPOINT ["node", "server.js"]
   CMD ["--port=3000"]
   ```
4. Reorder instructions for better caching if needed; rebuild.
5. Confirm the app still runs.

**Expected result:** Smaller/faster build; app unchanged.

> **Stop & think:** After adding `.dockerignore`, did the image get smaller, the build get faster, or both? Which did you expect, and why?

**Pitfall:** Without `.dockerignore`, `COPY . .` can drag in `node_modules`, `.git`, and a secret-filled `.env`. Add it **before** your first real build.

---

## Lab 8 — Multi-Stage Build
**~45 min · Goal:** Slim a fat image dramatically with a multi-stage build.

1. Build the single-stage image and record its size (`docker image ls`).
2. Rewrite the Dockerfile with a build stage and a runtime stage:
   ```dockerfile
   # ---- build ----
   FROM node:20 AS build
   WORKDIR /app
   COPY package*.json ./
   RUN npm ci
   COPY . .
   RUN npm run build

   # ---- run ----
   FROM nginx:alpine
   COPY --from=build /app/dist /usr/share/nginx/html
   ```
3. Build and compare sizes.
4. Run the final image and confirm it works.

**Expected result:** The final image is a fraction of the single-stage size.

> **Stop & think:** Your image dropped from hundreds of MB to tens. What exactly is **no longer** in it — and why does a smaller image also mean a smaller security attack surface?

**Pitfall:** Forgetting `--from=build` on the `COPY` — then nothing from the build stage reaches the final image and the app is missing its files.

---

## Lab 9 — Containerize the Course App
**~40 min · Goal:** Produce the production image you will use for the rest of the course.

1. Choose an appropriate **slim** base image for your language.
2. Structure for layer caching: dependencies before source.
3. Read all config from environment variables (no hard-coded URLs/secrets).
4. Use a multi-stage build if the app has a build/compile step.
5. Build, tag (`myapp:0.1`), run, and verify.

**Expected result:** A clean, runnable production image — your final-project starting point.

> **Stop & think:** This image is your deliverable on Day 3. What would make it (a) faster to rebuild and (b) safer to ship? Write it down now.

**Pitfall:** Baking environment-specific values (a prod database URL, an API key) into the image. Keep the image generic; inject config at run time.

---

## Lab 10 — Database with a Volume
**~50 min · Goal:** Run a real database and prove its data survives a container's death.

1. Run Postgres with a **named volume** for its data directory:
   ```bash
   $ docker volume create appdata
   $ docker run -d --name db -e POSTGRES_PASSWORD=secret \
       -v appdata:/var/lib/postgresql/data postgres:16
   ```
2. Create some data:
   ```bash
   $ docker exec -it db psql -U postgres -c "CREATE TABLE t(x int); INSERT INTO t VALUES (42);"
   ```
3. Destroy the container:
   ```bash
   $ docker rm -f db
   ```
4. Start a **new** container on the **same** volume:
   ```bash
   $ docker run -d --name db -e POSTGRES_PASSWORD=secret \
       -v appdata:/var/lib/postgresql/data postgres:16
   ```
5. Confirm the data survived:
   ```bash
   $ docker exec -it db psql -U postgres -c "SELECT * FROM t;"
   ```

**Expected result:** The `42` row is still there.

> **Stop & think:** You removed the container but kept the volume, and the data survived. Where does that volume actually live, and what would have happened with a **bind mount** to a host folder instead?

**Pitfall:** Mount on the **right path**. For Postgres it is `/var/lib/postgresql/data`. Mount the wrong path and nothing persists.

---

## Lab 11 — Multi-Container Network
**~55 min · Goal:** Wire two containers together so they talk **by name** on a private network.

1. Create a user-defined network:
   ```bash
   $ docker network create appnet
   ```
2. Run a db and an app on it:
   ```bash
   $ docker run -d --name db  --network appnet -e POSTGRES_PASSWORD=secret postgres:16
   $ docker run -dit --name api --network appnet alpine sh
   ```
3. From the app, reach the db **by name**:
   ```bash
   $ docker exec -it api sh
   / # nc -zv db 5432        # connects by the name "db"
   ```
4. Try the same on the **default** bridge (run both without `--network appnet`) and observe the difference — name resolution fails there.
5. Inspect who is attached:
   ```bash
   $ docker network inspect appnet
   ```

**Expected result:** Name resolution works on `appnet`, not on the default bridge.

> **Stop & think:** By **name** it worked on your network but not on the default bridge. What does a user-defined network give you that the default one does not?

**Pitfall:** Reaching for old `--link`. You do not need it. A user-defined network provides DNS automatically.

---

# Day 3 — Compose, Registry, CI/CD & Security

## Demo 3 (instructor) — The Compose stack
The instructor brings up a provided `compose.yaml` (app + db + cache) with `docker compose up`, opens the app, lists the auto-created network, tails one service's logs, then `docker compose down`.

> **Stop & think:** Compose created a network you never asked for. What is it called, which services are on it, and how does the app find the database without any IP addresses?

---

## Lab 12 — Extend the Stack
**~40 min · Goal:** Add the course app to a Compose stack and make startup robust.

1. Start from this `compose.yaml`:
   ```yaml
   services:
     api:
       build: .
       ports: ["3000:3000"]
       environment:
         DATABASE_URL: postgres://db:5432/app
       depends_on:
         db:
           condition: service_healthy
     db:
       image: postgres:16
       environment:
         POSTGRES_PASSWORD: secret
       healthcheck:
         test: ["CMD-SHELL", "pg_isready -U postgres"]
         interval: 5s
         retries: 5
   volumes:
     appdata: {}
   ```
2. Bring it up: `docker compose up --build`.
3. Confirm the api waits for the db to be **healthy** before starting.
4. Tail logs: `docker compose logs -f api`.
5. Tear down: `docker compose down`.

**Expected result:** Clean, ordered startup every time.

> **Stop & think:** Remove the `condition: service_healthy` and bring the stack up cold a few times. Does the api sometimes fail on first boot? Why does `service_healthy` fix it where plain `depends_on` does not?

**Pitfall:** Do **not** add a top-level `version:` key — it is obsolete in modern Compose and only produces a warning.

---

## Lab 13 — Tag & Push
**~40 min · Goal:** Publish your course-app image the way CI will.

1. Log in: `docker login`.
2. Tag with a semantic version under your namespace:
   ```bash
   $ docker tag myapp:0.1 levep79/myapp:1.0
   $ docker push levep79/myapp:1.0
   ```
3. Also tag with the short git SHA and push:
   ```bash
   $ docker tag myapp:0.1 levep79/myapp:$(git rev-parse --short HEAD)
   $ docker push levep79/myapp:$(git rev-parse --short HEAD)
   ```
4. Pull it back under a fresh name to verify:
   ```bash
   $ docker pull levep79/myapp:1.0
   ```

**Expected result:** Both tags appear in your registry; the pull succeeds.

> **Stop & think:** You pushed both `:1.0` and `:<gitsha>`. Six months from now, which tag tells you **exactly** which commit is running in production — and why is `:latest` not good enough?

**Pitfall:** The repository part of the tag **is** the destination. No registry prefix means Docker Hub. For a private registry, prefix the full host: `registry.company.com/team/myapp:1.0`.

---

## Lab 14 — Build–Test–Push Pipeline
**~50 min · Goal:** Wire a CI pipeline that builds your image, tests it, and pushes it.

1. Add a workflow (GitHub Actions example) at `.github/workflows/ci.yml`:
   ```yaml
   name: ci
   on: { push: { branches: [main] } }
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - run: docker build -t myapp:${{ github.sha }} .
         - run: docker run --rm myapp:${{ github.sha }} npm test
         - name: Push
           run: |
             echo "${{ secrets.REGISTRY_TOKEN }}" | docker login -u "${{ secrets.REGISTRY_USER }}" --password-stdin
             docker tag myapp:${{ github.sha }} levep79/myapp:${{ github.sha }}
             docker push levep79/myapp:${{ github.sha }}
   ```
2. Commit and push; watch the pipeline run.
3. Confirm the image appears in your registry, tagged with the SHA.
4. **Break a test on purpose**, push, and watch CI stop before the push step.

**Expected result:** Green pipeline pushes; a failing test blocks the push.

> **Stop & think:** You ran the tests **inside the image you are about to ship**, not against the host. Why does testing the actual artifact catch a class of bugs that testing the source alone never will?

**Pitfall:** Never hard-code registry credentials in the workflow. Use repository **secrets**.

---

## Lab 15 — Scan & Harden
**~45 min · Goal:** Find real vulnerabilities in your image and reduce them.

1. Get a summary:
   ```bash
   $ docker scout quickview myapp:1.0
   ```
2. List the actual CVEs and note the critical/high count:
   ```bash
   $ docker scout cves myapp:1.0
   ```
3. Add a non-root user to the Dockerfile:
   ```dockerfile
   COPY --chown=node:node . .
   USER node
   ```
4. Switch to a slimmer / newer base image and ask Scout for base-image fixes:
   ```bash
   $ docker scout recommendations myapp:1.0
   ```
5. Rebuild and re-scan; compare the numbers.

**Expected result:** A measurably lower CVE count and a non-root runtime.

> **Stop & think:** Swapping the base image changed your CVE count more than any app-code change could. What does that tell you about where most container risk actually comes from?

**Pitfall:** `docker scan` / the old Snyk integration is retired. Use **`docker scout`** (it ships with Docker).

---

## Lab 16 — Secrets & Least Privilege
**~35 min · Goal:** Take a hardened image further — keep a build secret out of the layers and lock down how the container runs.

> **Prerequisite:** start from your Lab 9 / Lab 15 production image.

1. Confirm the image runs as non-root (add `USER node` if not), and rebuild:
   ```bash
   $ docker build -t myapp:secure .
   $ docker run --rm myapp:secure whoami      # node, not root
   ```
2. Run with a read-only root filesystem plus a writable tmpfs for scratch space:
   ```bash
   $ docker run -d --name app --read-only --tmpfs /tmp \
       -p 8080:3000 myapp:secure
   $ curl localhost:8080                        # still works
   ```
3. Drop all Linux capabilities and block privilege escalation:
   ```bash
   $ docker rm -f app
   $ docker run -d --name app --read-only --tmpfs /tmp \
       --cap-drop ALL --security-opt=no-new-privileges \
       -p 8080:3000 myapp:secure
   ```
4. Replace a COPYed token with a **BuildKit secret** (no secret in any layer):
   ```dockerfile
   # syntax=docker/dockerfile:1
   RUN --mount=type=secret,id=token \
       TOKEN=$(cat /run/secrets/token) ./fetch-deps.sh
   ```
   ```bash
   $ docker build --secret id=token,src=./token.txt -t myapp:secure .
   $ docker history myapp:secure | grep -i token    # nothing leaks
   ```
5. Scan and produce an SBOM:
   ```bash
   $ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
       -v ~/trivy-cache:/root/.cache/trivy \
       aquasec/trivy image --severity HIGH,CRITICAL myapp:secure
   $ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
       aquasec/trivy image --format cyclonedx -o /dev/stdout myapp:secure > sbom.json
   ```

**Expected result:** A non-root, read-only, capability-dropped container that still
serves traffic, with no secret recoverable from the image and an SBOM on disk.

> **Stop & think:** You dropped ALL capabilities and added none back, yet the app
> still ran. What does that tell you about how many privileges it actually needed —
> and what would you add back only if a feature broke?

**Pitfall:** `--read-only` breaks apps that write to disk (temp files, caches, the
app's own logs). The fix isn't to drop `--read-only` — it's to mount a `--tmpfs`
or a volume on exactly the paths that need to be writable.

---

## Lab 17 — Observe & Troubleshoot a Stack
**~30 min · Goal:** Diagnose a misbehaving multi-container app with a method, not a guess.

> **Prerequisite:** the broken Compose stack (lives in `demos/troubleshoot/`). It
> brings up an app that can't reach its database.

1. Bring it up and watch it fail:
   ```bash
   $ docker compose up -d
   $ docker compose ps          # which service is unhealthy / restarting?
   ```
2. Read the logs of the failing service first:
   ```bash
   $ docker compose logs app
   ```
3. While it misbehaves, watch the daemon and resource usage:
   ```bash
   $ docker events &            # live event stream
   $ docker stats               # CPU / memory per container
   ```
4. Inspect the suspect — env, network, health:
   ```bash
   $ docker compose exec app sh -c 'env | grep DATABASE'
   $ docker inspect -f '{{json .State.Health}}' <db-container>
   ```
5. Change **one** thing (a wrong hostname, a credential mismatch, a missing
   healthcheck condition), bring it back up, and verify. State the root cause in one
   sentence.

**Expected result:** The stack comes up healthy, and you can name the root cause
precisely.

> **Stop & think:** The same three commands — `ps`, `logs`, `inspect` — cracked
> this even though it's a multi-service stack. Why are they always your opening
> moves, and in that order?

**Pitfall:** Changing several things at once. You'll fix it but won't know which
change did it — and you won't have learned anything transferable. One change,
verify, repeat.

---

## Demo 4 (instructor) — Diagnose together
As a group, work through several broken scenarios (a container that exits, one that can't be reached, a slow build, a huge image). For each: **predict the cause first**, then confirm with `docker ps -a`, `docker logs`, and `docker inspect`.

> **Stop & think:** Across all of these, the same three commands diagnosed almost everything. Which three — and why are they always your opening moves?

---

## Final Project — Ship the Course App End-to-End
**~60 min · Individual, hands-on**

Bring together everything from the week on your course app.

**Deliverables:**
1. A **multi-stage, non-root** Dockerfile.
2. A `compose.yaml` with **app + database + healthcheck** and ordered startup.
3. The image **built, tagged** (semver **and** git SHA) and **pushed** to your registry.
4. A `docker scout` scan with the **worst findings addressed**.
5. A **CI workflow**: build → test → push.

**Done means:** `docker compose up` brings the whole app up cleanly from scratch, the image is in your registry under a meaningful tag, the scan is clean of criticals, and a push to `main` runs your pipeline green.

> **Stop & think:** This is the whole real-world loop: **build → compose → ship → scan → automate**. Where in *your* project would this fit, and what is the first piece you would adopt on Monday?

---

## Where to go next
- **Orchestration:** Kubernetes — running these images across many machines.
- **BuildKit power features:** cache mounts, secret mounts, multi-arch with `buildx`.
- **Supply-chain security:** SBOMs, signing, and `docker scout` as a CI gate.
- **Compose depth:** profiles, `include`, and `docker compose watch` for fast inner-loop dev.

*Solutions for every lab are on the `solutions` branch of the course repo.*