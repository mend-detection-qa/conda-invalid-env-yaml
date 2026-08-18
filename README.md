# conda-invalid-env-yaml

**Regression probe for [TKA-10917](https://mend-io.atlassian.net/browse/TKA-10917)**

## What this tests

The Mend Conda resolver must **not** treat a file named `environment.yaml` or `environment.yml` as a Conda manifest unless the file contains at least one Conda schema key (`name:`, `channels:`, or `dependencies:`).

This repo contains:
- `application/src/main/resources/environment.yaml` — a **Hippo CMS / JCR** configuration file with **zero Conda schema keys**
- `pom.xml` — a standard Maven manifest (the real package manager for this project)

## Expected scanner behaviour (post-fix)

| Check | Expected |
|---|---|
| Conda dependencies detected | **0** |
| `[manifest] of [conda]` log line for the JCR file | **absent** |
| `EnvironmentSectionNotValid` in scanner log | **absent** |
| `CondaDependencyResolver tree size` log line | **absent** |
| Maven dependencies detected | ≥ 2 (commons-lang3, guava) |

## Bug summary

Agent v26.6.1/26.6.2 (Mend for GHE) invoked `conda env create --file application/src/main/resources/environment.yaml` against the JCR file. Conda emitted `EnvironmentSectionNotValid` but exited 0. With `.condarc` `create_default_packages` configured in Capital One's scanner container, 34 phantom dependencies (pip, blas=*=openblas) were reported as project dependencies, generating spurious Critical/High CVE findings against a pure Java repo.

## Coverage pattern

`invalid-non-conda-yaml` — catalogued in `python-conda/feature-coverage-patterns.md`.
