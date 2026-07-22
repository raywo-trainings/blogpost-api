# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## What this repository is

An API-first design project (part of an API training) that specifies a
**Blogpost API** as an OpenAPI 3.1.0 document. The entire contract lives in
`blogposts.yaml`. There is currently no implementation, server, build system,
dependencies, or tests — the work here is authoring the API specification
itself.

## Working with the spec

- `blogposts.yaml` is the single source of truth. It follows the OpenAPI 3.1.0
  structure (`info`, `servers`, `paths`, and — as it grows — `components`).
- The `paths` section is where endpoints are added. Define reusable schemas
  under `components/schemas` and reference them with `$ref` rather than inlining
  duplicated shapes.
- Keep the document valid OpenAPI 3.1.0. When adding operations, verify the YAML
  parses and conforms to the spec (e.g. via an editor's OpenAPI plugin,
  `redocly lint blogposts.yaml`, or `swagger-cli validate blogposts.yaml`)
  before considering a change complete.

## Notes

- This is an IntelliJ IDEA project (`.idea/`, `BlogPost API.iml`); the module is
  a plain general module with no language SDK configured.
- The declared server URL is `https://localhost:8080` — a placeholder for a
  future implementation, not a running service.
