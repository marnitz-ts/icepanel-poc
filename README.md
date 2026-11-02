# IcePanel architecture discovery

This repository is a read-only analysis project. It studies selected company repositories and records accurate software architecture with [IcePanel](https://icepanel.io/) and the [C4 model](https://c4model.com/).

Company source stays unchanged. Findings, models, and diagrams are written only in this project.

## Purpose

- Discover what the selected repositories actually contain.
- Separate larger applications from independently deployable services.
- Record integrations only when source evidence supports them.
- Model confirmed structure as C4 context, container, and component views for IcePanel.

Company repositories have not been analysed yet.

## Layout

| Path | Contents |
| --- | --- |
| `analysis/inventory` | Which repositories are in scope, and why |
| `analysis/repositories` | Notes taken from each repository |
| `analysis/services` | Applications and independently deployable services |
| `analysis/integrations` | Connections, each with evidence and a confidence label |
| `architecture/models` | C4 model definitions for IcePanel |
| `architecture/diagrams` | Diagram exports and supporting views |
| `architecture/reviews` | Checks of models against source |
| `docs` | Project documentation |

## Workflow

1. Record selected repositories in `analysis/inventory`. Inspect company source in place. Do not edit it.
2. Write repository notes in `analysis/repositories`. Skip build output, dependencies, caches, and generated files. Do not copy secrets, credentials, tokens, or private keys.
3. Describe services in `analysis/services`. A shared library is not a deployed service. A larger application is not a set of independently deployable services unless deployment evidence says so. Identify legacy systems without treating them as deprecated unless the source says they are.
4. Record integrations in `analysis/integrations`. Cite the source file for every proposed connection. Label each finding **confirmed** (code evidence), **inferred**, or **unknown**.
5. Build C4 models in `architecture/models` and diagrams in `architecture/diagrams` from those records only. Do not add services, APIs, connections, or dependencies the analysis does not support.
6. Review each model in `architecture/reviews` before treating a diagram as accurate.

## Agent rule

Cursor applies `.cursor/rules/architecture-analysis.mdc` to every session in this repository.
