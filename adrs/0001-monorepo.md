---
description: Why we chose a monorepo structure
---

# ADR 0001: Monorepo Structure

## Status

Accepted

## Date

2025-06-15

## Context

The Sensei platform comprises multiple service components: the API gateway (TypeScript), the verification engine kora-core (Rust), ML inference services (Python), shared protocol buffers, client SDKs (TypeScript, Python, Go, Rust), CLI tooling, infrastructure-as-code definitions, and documentation. These components have significant interdependencies: the API layer depends on type definitions shared with the SDKs, the verification engine consumes protocol buffer schemas also used by the ML services, and integration tests require coordinated versioning across all services.

The team evaluated two structural approaches:

**Polyrepo.** Each component in its own repository. Independent versioning, independent CI/CD pipelines, independent access control. This model is well-suited to organizations with large, independent teams working on loosely coupled services.

**Monorepo.** All components in a single repository with workspace-based dependency management. Shared CI/CD infrastructure, atomic cross-component changes, unified versioning for coordinated releases.

At the current stage of the project (early product development, team of 3-8 engineers), the tradeoffs favor monorepo.

## Decision

Adopt a monorepo structure using **pnpm workspaces** for package management, with the following layout:

```text
sensei-ai/
  packages/
    api/              # TypeScript API gateway
    kora-core/        # Rust verification engine
    ml-services/      # Python ML inference
    sdk-typescript/   # TypeScript SDK
    sdk-python/       # Python SDK
    cli/              # CLI tool
    shared/           # Shared types, proto definitions
  infra/              # Terraform, Docker Compose
  docs/               # Documentation
  scripts/            # Build and CI tooling
```

pnpm is selected over npm and yarn for its strict dependency resolution, disk-efficient content-addressable storage, and native workspace support. Rust and Python components are managed by their respective toolchains (Cargo, Poetry) but integrated into the monorepo CI pipeline via shared scripts.

## Consequences

**Benefits:**

- Atomic commits across components. A change to a shared protocol buffer definition, the API handler that uses it, and the SDK that exposes it can land as a single commit with a single review.
- Simplified dependency management. Internal packages reference each other via workspace protocol (`workspace:*`), eliminating version drift between components.
- Unified CI/CD. A single pipeline builds, tests, and deploys all components, with change detection to skip unaffected packages.
- Reduced onboarding friction. New engineers clone one repository and have the entire system available for local development.

**Drawbacks:**

- Repository size will grow over time. Mitigated by `.gitattributes` configuration for large files and sparse checkout support for contributors who work on a single component.
- CI pipeline complexity increases with heterogeneous toolchains (pnpm, Cargo, Poetry). Mitigated by per-package build scripts invoked from a shared CI configuration.
- Access control is coarser-grained than polyrepo. All contributors can view all code. For the current team size, this is acceptable and even desirable; it may require revisiting if the organization scales beyond 20-30 engineers.

**Revisiting this decision:** If the team grows beyond 30 engineers or if CI pipeline duration exceeds 15 minutes with change detection enabled, the monorepo structure should be re-evaluated. A migration path to polyrepo is preserved by maintaining clear package boundaries and explicit internal APIs.

## References

- [pnpm Workspaces](https://pnpm.io/workspaces)
- Google monorepo paper: "Why Google Stores Billions of Lines of Code in a Single Repository" (2016)
- ADR format: Michael Nygard, "Documenting Architecture Decisions" (2011)
