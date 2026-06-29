# Docker & Containers for Developers

**NAYA College · 3-Day Practical Course · Docker 24+**

A hands-on Docker course that takes **one application** from a single container to a
CI pipeline that builds, tests, scans, and ships it. This repo holds everything for
the three audiences: students (clone and work), the instructor (publish images, run
demos), and the reference answers.

## Layout

```
docker-for-developers/
├── docs/lab-guide.md        # the lab guide — 22 labs, 4 demos, final project
├── app/                     # THE course app (source only — you write the Dockerfile)
├── lab-images/              # images the INSTRUCTOR publishes once (broken-demo, course-app)
├── demos/                   # instructor-led demo material
│   ├── demo-03-compose/     # Demo 3 — app + db + cache stack
│   └── troubleshoot/        # Lab 22 — deliberately broken stack to diagnose
└── solutions/               # answers — Day 1 answer key + a folder per Day 2–3 lab
    └── Labs Solutions Full/  # instructor edition: full guide WITH solutions inline
```

## Getting started (students)

1. Check your setup (see `docs/lab-guide.md` → Pre-flight checklist):
   ```bash
   docker version && docker compose version && docker run hello-world
   ```
2. Open **`docs/lab-guide.md`** and work the labs in order. The app you build lives in
   `app/`.

## Instructor prep (before the course)

1. Publish the two lab images — see `lab-images/README.md`
   (`levep79/broken-demo` for Lab 5, `levep79/course-app` for Lab 6).
2. Have the demo stacks ready in `demos/`.

## How solutions work

`solutions/` is a normal folder in this repo — answers are openly available
(this course trusts learners to attempt each lab first; the value is in trying before
peeking).

- **Day 1** (Labs 1–10) is CLI-only → `solutions/day-1/` is an answer key (expected
  output + resolved Stop & think). No solution files, by design.
- **Days 2–3** → one folder per lab (`solutions/lab-11-build/` … `lab-22-troubleshoot/`,
  plus `final-project/`) with the finished artifact(s).
- **`solutions/Labs Solutions Full/`** → an instructor edition of the lab guide with
  every solution and Stop & think answer inline.

Compare rather than copy when you're stuck:
```bash
diff app/Dockerfile solutions/lab-13-multistage/Dockerfile
```

> If you ever want to *withhold* answers until after each day, the cleanest options are
> to keep `solutions/` on a separate branch you publish later, or in a separate
> access-controlled repo. As shipped, everything is on one branch and openly readable.

## Tooling notes

- Use **`docker compose`** (v2, the Go plugin) — not the retired hyphenated `docker-compose`.
- Image scanning uses **Trivy** (runs as a container, nothing to install). Docker Scout
  ships only with Docker Desktop; if you have it, the equivalents are noted in Lab 20.
- `compose.yaml` files have no top-level `version:` key (obsolete in modern Compose).

## Course structure

- **Day 1 — Foundations & the CLI:** containers vs VMs, architecture, Linux basics,
  the CLI, consuming images, run mastery, logs/inspect/diff, housekeeping, capstone.
- **Day 2 — Images & Data:** images & layers, Dockerfiles, multi-stage, volumes &
  databases, networking.
- **Day 3 — Ship It:** Compose, registry, CI/CD, security & scanning, troubleshooting,
  final project.
