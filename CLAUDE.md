# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## What this repository is

An API-first design project (part of an API training) that specifies a
**Blogpost API** as an OpenAPI 3.0.1 document. The entire contract lives in
`blogposts.yaml`. There is currently no implementation, server, build system,
dependencies, or tests — the work here is authoring the API specification
itself.

## Working with the spec

- `blogposts.yaml` is the single source of truth. It follows the OpenAPI 3.0.1
  structure (`info`, `servers`, `paths`, and — as it grows — `components`).
- The `paths` section is where endpoints are added. Define reusable schemas
  under `components/schemas` and reference them with `$ref` rather than inlining
  duplicated shapes.
- Keep the document valid OpenAPI **3.0.x**. In particular, this means using
  `nullable: true` (not `type` null-unions) and `enum: [VALUE]` for
  single-value constants (the `const` keyword is JSON-Schema 2020-12 and only
  valid in OpenAPI 3.1).
- When adding operations, verify the YAML parses and conforms to the spec
  (e.g. via an editor's OpenAPI plugin, `redocly lint blogposts.yaml`, or
  `swagger-cli validate blogposts.yaml`) before considering a change complete.

## Branches

The training work lives on separate branches that are **intentionally not
merged** into `main`, so the API's stepwise evolution stays traceable. `main`
therefore holds only the base Blogpost resource — do not assume it reflects the
full API.

- `authors`, `hashtags`, `ratings` — each branches directly off `main` and adds
  a single resource/feature.
- `all-together` — merges the three feature branches into one spec.
- `all-together-with-problemdetails` — builds on `all-together` and adds
  unified RFC-9457 error handling (`ProblemDetail`, `application/problem+json`,
  400 vs. 422) demonstrating `allOf`/`anyOf`.

When asked to work on a feature, check out the relevant branch first. See
`README.md` for the full overview.

## Notes

- This is an IntelliJ IDEA project (`.idea/`, `BlogPost API.iml`); the module is
  a plain general module with no language SDK configured.
- The declared server URL is `https://localhost:8080` — a placeholder for a
  future implementation, not a running service.
