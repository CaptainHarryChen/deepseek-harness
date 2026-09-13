# Agent Note: Harness-bridged catalog models

Status: implemented

English | [中文](2026-09-13-harness-bridged-catalog-models.zh.md)

## Problem

The `opencode-go` route in the installed pi-ai catalog serves release-cadence model ids: the endpoint added `deepseek-v4.1-flash` (DeepSeek V4.1 Flash, 1M context / 384K max output) while the installed `@earendil-works/pi-ai` catalog — 0.85.1, the latest published version — still knows only `deepseek-v4-flash`, `deepseek-v4-pro`, and `deepseek-v4-flash-vision-exp` for the route. The harness therefore offered no selectable option for a model the endpoint already served.

## Decision

`dsh-llm-pi-ai` bridges a small set of catalog gaps inside its own catalog integration (`CATALOG_ADDITIONS` in `src/catalog.ts`): keyed by route and missing model id, each entry names the sibling entry of the same route it is cloned from, so the wire protocol, compat switches, thinking map, capacities, and pricing metadata ride along, and only the id and display name change. The bridge adds `opencode-go/deepseek-v4.1-flash` (cloned from `deepseek-v4-flash`) to `catalogModels()`, which feeds model discovery, model resolution, and selector surfaces. A model the installed catalog already ships is left to the catalog's own entry, which is also the upstreamed signal: drop the bridge entry, its merge branches, and its pinned tests when the installed catalog carries the id.

## Alternatives considered

**pnpm-patch of pi-ai's shipped catalog data.** Patching `dist/providers/data/opencode-go.json` would work, but it edits a third-party package's payload invisible to harness source, churns the lockfile and patch plumbing, and collides with the next pi-ai bump. The harness already owns the catalog integration and its drift gates, so a narrow harness-side bridge lives in the same home as the route facts the opencode-go session-header fix already added.

**Configuration-only (`models` / `modelOverrides`).** Requires every deployment to hand-write the entry; the model should be selectable like its catalog siblings without a settings edit, and nothing here ships a default configuration.

**Bumping pi-ai.** The repo already treats fresh pi-ai releases as the normal path for catalog updates, but 0.85.1 is the latest published release and no newer version carries the id, so there is nothing to bump to.

## Consequences

The OpenCode Go route offers `deepseek-v4.1-flash` wherever the catalog is read, with wire behavior inherited from `deepseek-v4-flash` until pi-ai ships a distinct entry. Keeping the bridge current is a small explicit maintenance duty: the moment the installed catalog carries the id, the catalog's own entry wins automatically and the bridge entry, its merge branches, and its pinned tests are removed.

## Related

The [provider-routed LLM adapters note](2026-07-14-provider-routed-llm-adapters.md) owns the route/catalog design this bridge extends.