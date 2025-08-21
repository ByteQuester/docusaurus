---
id: release-process
slug: /reference/release-process
sidebar_position: 18
---

# Release Process

:::info This document outlines our release process, including semantic versioning, changelogs, promotion, and rollback procedures. :::

## Semantic Versioning

We use [Semantic Versioning](https://semver.org/) (SemVer) for our releases. This means that our version numbers are in the format of `MAJOR.MINOR.PATCH`.

- **MAJOR** version when you make incompatible API changes.
- **MINOR** version when you add functionality in a backward-compatible manner.
- **PATCH** version when you make backward-compatible bug fixes.

## Changelogs

Changelogs are automatically generated from [Conventional Commits](https://www.conventionalcommits.org/). This means that our commit messages follow a specific format, which allows us to automatically generate a changelog for each release.

## Promotion Flow

For details on how to promote changes from development to production, see the [Promotion Flow](./promotion-flow.md) document.

## Rollback Procedure

To roll back a release, you can simply revert the pull request that promoted the change. Once the revert pull request is merged, Argo CD will automatically detect the change in the Git repository and sync the previous version of the application to the production environment.
