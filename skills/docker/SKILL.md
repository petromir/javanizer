---
name: docker
description: Expert guidance for building optimal, secure, and production-ready Docker images. Use this when creating or refactoring Dockerfiles to improve build speed, reduce image size, and enhance security.
---

# Docker Optimization

This skill provides expert procedural guidance for building Docker images that are optimized for performance, security, and maintainability.

## Core Principles

-   **Minimize Image Size**: Use small base images (Alpine, Slim, Distroless) and multi-stage builds.
-   **Optimize Layer Caching**: Arrange instructions from least to most frequent changes.
-   **Security First**: Run as a non-root user, use specific tags, and scan for vulnerabilities.
-   **Maintainability**: Use clear, readable instructions and labels.

## References

| Category           | When to read                   | File path                                        |
|--------------------|--------------------------------|--------------------------------------------------|
| Dockerfile         | Creating, updating Dockerfiles | [dockerfile](references/dockerfile.md)           |
| Security & Runtime | Improving security and runtime | [dockerfile](references/security-and-runtime.md) |

