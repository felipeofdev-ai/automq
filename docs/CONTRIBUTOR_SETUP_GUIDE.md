# AutoMQ Contributor Setup Guide

> A beginner-friendly guide from fork to first Pull Request.  
> Written from the perspective of a first-time contributor.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Fork & Clone](#fork--clone)
- [Start Local Environment](#start-local-environment)
- [Build from Source](#build-from-source)
- [Key Modules](#key-modules)
- [Contribution Workflow](#contribution-workflow)
- [Community & Support](#community--support)
- [Commands Cheat Sheet](#commands-cheat-sheet)

---

## Prerequisites

Ensure all of the following are in place **before** cloning the repository.  
Missing any of these is the most common source of friction for new contributors.

| Requirement | How to Verify |
|---|---|
| Java 17+ | `java -version` |
| Docker 20.x+ | `docker --version` |
| Docker Compose v2 | `docker compose version` (note: `compose`, not `compose-v1`) |
| 4 GB RAM allocated to Docker | Docker Desktop → Settings → Resources |
| Port 9092 free | `lsof -i :9092` (should return empty) |
| Port 9000 free | `lsof -i :9000` (should return empty) |

> **Note:** The Quick Start environment is for **evaluation only** — a single node with MinIO simulating S3.  
> Do not use it for performance benchmarks or production assessments.

---

## Fork & Clone

**1. Fork the repository**

Go to [https://github.com/AutoMQ/automq](https://github.com/AutoMQ/automq) and click **Fork**.

**2. Clone your fork**

```bash
git clone https://github.com/<your-username>/automq.git
cd automq
```

**3. Add the upstream remote**

This lets you sync your fork with the main project later.

```bash
git remote add upstream https://github.com/AutoMQ/automq.git
```

**4. Verify your remotes**

```bash
git remote -v
# origin    https://github.com/<your-username>/automq.git (fetch)
# upstream  https://github.com/AutoMQ/automq.git (fetch)
```

---

## Start Local Environment

### Download the Docker Compose file

```bash
curl -O https://raw.githubusercontent.com/AutoMQ/automq/refs/tags/1.5.5/docker/docker-compose.yaml
```

This pulls a pre-configured file that starts one AutoMQ node and one MinIO container (S3-compatible storage).

### Start the cluster

```bash
docker compose -f docker-compose.yaml up -d
```

Wait ~30 seconds for services to initialize, then verify:

```bash
docker compose -f docker-compose.yaml ps
# All services should show status: running
```

### Verify with a producer test

Run a quick producer test inside the Docker network to confirm the broker is accepting connections:

```bash
docker run --network automq_net automqinc/automq:latest /bin/bash -c \
  "/opt/automq/kafka/bin/kafka-producer-perf-test.sh \
  --topic test-topic \
  --num-records=1024000 \
  --throughput 5120 \
  --record-size 1024 \
  --producer-props bootstrap.servers=server1:9092 linger.ms=100 batch.size=524288 buffer.memory=134217728"
```

You should see throughput metrics printed to stdout. If you see connection errors, check that port 9092 is free.

### Tear down

Always stop and remove containers after your session to free resources:

```bash
docker compose -f docker-compose.yaml down
```

> **Tip:** For multi-node cluster testing, use `docker/docker-compose-cluster.yaml` instead — three AutoMQ nodes for testing partition rebalancing and failover.

---

## Build from Source

AutoMQ uses the **Gradle wrapper** — never install Gradle system-wide. Always use `./gradlew`.

### Full build (skip tests for speed on first run)

```bash
./gradlew build -x test
```

Initial build takes 5–15 minutes depending on your machine and network.

### Run all tests

```bash
./gradlew test
```

### Run tests for a specific module

Faster for iterative development:

```bash
./gradlew :s3stream:test
```

---

## Key Modules

| Module | Description |
|---|---|
| `s3stream/` | Core S3 storage engine. WAL, object storage, caching. The heart of what makes AutoMQ different from Kafka. |
| `core/` | Kafka broker core, forked from Apache Kafka. Contains the main broker logic. |
| `automq-metrics/` | Prometheus and OpenTelemetry metrics export. Good entry point for observability contributions. |
| `automq-shell/` | CLI tooling for cluster management operations. |
| `docker/` | Docker Compose setups for local dev and multi-node testing. |
| `examples/` | Usage examples and integration demos. Ideal target for a first contribution. |

---

## Contribution Workflow

Every contribution — code, documentation, test, or example — follows the same process.  
**Do not skip steps.** PRs that bypass this process are closed without review.

### 1. Find or create an issue

Browse [open issues](https://github.com/AutoMQ/automq/issues). Look for issues tagged **`good first issue`** as your entry point.  
If none are available, open a new issue using the provided template **before writing any code**.

### 2. Claim the issue

Comment `/assign` on the issue thread. The GitHub bot will assign it to you automatically.  
This prevents duplicate work across contributors.

### 3. Create a feature branch

Never work on `main` directly. Always branch from the latest upstream:

```bash
git fetch upstream
git checkout -b feat/your-feature-name upstream/main
```

### 4. Make your changes

Follow the existing code style. The project enforces **Checkstyle** — the build will fail if style rules are violated.  
If adding a new feature, include corresponding unit tests.

### 5. Sync with upstream before opening a PR

```bash
git fetch upstream
git rebase upstream/main
```

### 6. Open a Pull Request

Push your branch and open a PR against `AutoMQ/automq:main`.  
The repository includes a **PR template** — fill it out completely. Incomplete descriptions delay review.

### 7. Sign the CLA

AutoMQ requires a Contributor License Agreement. You will be prompted automatically on your first PR.  
PRs cannot be merged without a signed CLA.

### 8. Respond to review feedback

Maintainers review PRs regularly. Respond promptly to comments.  
Push new commits to address feedback — **do not force-push after review has started**.  
PRs without updates are automatically closed after a period of inactivity.

---

## Community & Support

Use the right channel for the right type of question.

| Channel | Best Used For |
|---|---|
| [GitHub Issues](https://github.com/AutoMQ/automq/issues) | Bug reports, feature requests, scoped technical questions. Include repro steps and environment details. |
| [Community Slack](https://go.automq.com/slack) | Real-time discussion, quick questions, design ideas, and introducing yourself. |
| WeChat Group | Active in APAC timezone. See the README for the QR code. |
| [Gurubase AI](https://gurubase.io/g/automq) | Ask questions about the AutoMQ codebase before escalating to maintainers. |
| [DeepWiki](https://deepwiki.com/AutoMQ/automq) | Auto-generated codebase documentation. Useful for architecture exploration. |

### Getting unblocked efficiently

Before asking for help, check in this order:

1. Search **closed GitHub Issues** — your problem has likely been encountered before.
2. Search the **Slack archive** — many common setup questions are already answered.
3. Use **DeepWiki** or **Gurubase** for codebase navigation questions.
4. Ask in Slack `#general` with full context: what you tried, what you expected, what happened.

---

## Commands Cheat Sheet

| Action | Command |
|---|---|
| Start local cluster | `docker compose -f docker-compose.yaml up -d` |
| Stop local cluster | `docker compose -f docker-compose.yaml down` |
| Check cluster status | `docker compose -f docker-compose.yaml ps` |
| Full build (no tests) | `./gradlew build -x test` |
| Run all tests | `./gradlew test` |
| Run module tests | `./gradlew :s3stream:test` |
| Sync with upstream | `git fetch upstream && git rebase upstream/main` |
| Claim an issue | Comment `/assign` on the GitHub issue thread |
| Create feature branch | `git checkout -b feat/your-feature-name upstream/main` |

---

## Further Reading

- [AutoMQ Documentation](https://www.automq.com/docs)
- [CONTRIBUTING_GUIDE.md](../CONTRIBUTING_GUIDE.md)
- [CODE_OF_CONDUCT.md](../CODE_OF_CONDUCT.md)
- [Architecture Overview](https://www.automq.com/docs/automq/architecture/overview)
- [DeepWiki — AutoMQ Codebase](https://deepwiki.com/AutoMQ/automq)

---

*AutoMQ is Apache 2.0 licensed. All contributors are expected to follow the [Code of Conduct](../CODE_OF_CONDUCT.md).*
