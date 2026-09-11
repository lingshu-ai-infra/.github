# Contributing to LingShu AI Infra

Thank you for your interest in contributing to LingShu AI Infra! 🎉

This document describes how to file issues, propose changes, and submit pull
requests in our repos.

## 📋 Code of Conduct

All participants are expected to follow our
[Code of Conduct](./lingshu-design/blob/main/CODE_OF_CONDUCT.md).
Please read it before participating.

## 🐛 Filing Issues

Before opening an issue:

1. **Search existing issues** to avoid duplicates.
2. **Use the issue template** that best fits your report (bug / feature /
   question).
3. **Provide minimal reproduction** — for bugs, include config and logs;
   for features, describe the use case clearly.

Good issue = minimal reproducible example + observed vs expected behavior +
environment (OS, Java version, cluster topology, GPU model).

## 🌿 Branching & Workflow

- `main` is the always-stable branch; PRs target `main`.
- Branch naming: `feat/<scope>-<short-desc>` / `fix/<scope>-<short-desc>` /
  `docs/<scope>-<short-desc>` / `chore/<scope>-<short-desc>`
- One logical change per PR (atomic commits, no squashed history).
- Rebase before merge; linear history preferred.

## ✍️ Commit Messages

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>

<footer>
```

Common types: `feat` / `fix` / `docs` / `style` / `refactor` / `test` /
`chore` / `perf`.

Examples:

- `feat(scheduler): add same-node soft affinity scoring`
- `fix(client): handle reconnect during in-flight query`
- `docs(design): clarify ControllerManager split-brain recovery`

## 🔧 Development Setup

Prerequisites:

- JDK 17+
- Maven 3.8+
- Docker (for integration tests)
- A working GPU node (RTX 3090 / A100 / V100 / T4) for end-to-end verification

Clone and build the prototype bundle:

```bash
git clone https://github.com/lingshu-ai-infra/lingshu-design
git clone https://github.com/lingshu-ai-infra/lingshu-gpu-client
git clone https://github.com/lingshu-ai-infra/lingshu-gpu-scheduler
git clone https://github.com/lingshu-ai-infra/lingshu-gpu-worker
mvn -pl lingshu-gpu-client,lingshu-gpu-scheduler,lingshu-gpu-worker -am clean install
```

Run the E2E demo:

```bash
cd lingshu-gpu-demo
mvn spring-boot:run
# in another terminal:
curl -X POST localhost:8080/demo/predict \
  -H 'Content-Type: application/json' \
  -d '{"text":"灵枢调度真的很强"}'
```

## 🧪 Testing

- Unit tests live next to source: `src/test/java/<package>/`.
- Integration tests under `src/test/java/<package>/integration/`.
- Coverage target: 80%+ on critical paths (codec, scheduler, worker
  executor).
- CI runs `mvn verify` on every PR — please run it locally first.

```bash
mvn -pl lingshu-gpu-scheduler test
mvn -pl lingshu-gpu-scheduler verify -Pit
```

## 📐 Architecture Conventions

- **Protobuf first** — any new cross-process message MUST be defined in
  `lingshu-gpu-proto` first, generated, then consumed.
- **No tight coupling** — components interact via Protobuf over Netty,
  not direct method calls.
- **Idempotency keys** required on all task-submission paths.
- **State machine purity** — task state transitions only via
  `TaskStateMachine.transition(taskId, newState, event)`.
- **No raw exceptions cross the wire** — always map to `ErrorCode` enum.

## 🎨 Style

- Lombok for boilerplate (`@Slf4j`, `@Data`, `@Builder`).
- Chinese is acceptable in comments and Javadoc; English for public API
  Javadoc.
- 4-space indent for Java; 2-space for YAML / JSON.
- No wildcard imports.
- Each public class / interface MUST carry `@author <github-handle>` line.

## 📜 License

By contributing, you agree that your contributions will be licensed under the
[Apache License 2.0](./lingshu-website/blob/main/LICENSE).