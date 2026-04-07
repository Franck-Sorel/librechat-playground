## Overview

This repo contains a minimal LibreChat Helm deployment that restores the normal custom-model UI for `Camer AI`, plus an experimental overlay that attempts to expose both the normal `Camer AI` model picker and MoE model specs together.

Use:

- `values.yaml` for the deployable base configuration
- `values-camer-ai-moe.experimental.yaml` for an experimental combined `Camer AI` + MoE UI overlay

The base deployment already includes:

- MongoDB connection and Meilisearch integration
- A custom OpenAI-compatible endpoint named `Camer AI`
- File upload limits for assistants
- Basic agent and assistant runtime settings
- Title generation and summarization settings compatible with the configured models

The experimental MoE overlay contains:

- `modelSpecs` entries for an orchestrator and specialists
- Specialist routing presets mapped to the models already exposed by `Camer AI`
- Explicit interface flags to keep the normal selector controls enabled
- MoE model specs grouped under `Camer AI`

## Files

- [values.yaml](/home/virtualhost/librechat/values.yaml): base LibreChat deployment
- [values-camer-ai-moe.experimental.yaml](/home/virtualhost/librechat/values-camer-ai-moe.experimental.yaml): experimental overlay that aims to keep the `Camer AI` picker while adding MoE specs
- [librechat-credentials-env.example.yaml](/home/virtualhost/librechat/librechat-credentials-env.example.yaml): example application secret manifest
- [librechat-api-key-secret.example.yaml](/home/virtualhost/librechat/librechat-api-key-secret.example.yaml): example API key secret manifest

## What We Configured

### Base LibreChat

- Disabled bundled auth for MongoDB and connected LibreChat to the external Mongo service
- Enabled Meilisearch support
- Added `Camer AI` as a custom OpenAI-compatible endpoint at `https://api.ai.camer.digital/v1`
- Configured the available model list explicitly
- Switched title and summary generation to `qwen3-8b` because it exists in the configured model catalog
- Added safe defaults for assistants, agent recursion, file size limits, and upload rate limits

### Experimental MoE

The MoE configuration is intentionally separate from the base deployment.

It defines these presets:

- `moe-orchestrator`
- `moe-strategy`
- `moe-creative`
- `moe-analysis`
- `moe-vision`
- `moe-fast`

Each preset uses one of the models already available in the `Camer AI` endpoint, so there is no second provider to configure.

## What LibreChat Docs Changed In Our Understanding

Using the LibreChat documentation from Context7, the important constraints are:

- Custom model choices in the normal chat UI come from `endpoints.custom.models`.
- `endpoints.agents` does not define chat models. It configures the separate Agents feature such as builder availability, recursion limits, citations, and capabilities.
- `endpoints.assistants` does not define chat models either. It configures the separate Assistants feature such as builder availability and polling/timeouts.
- LibreChat exposes configured models through `/api/models`, which is driven by configured endpoints.
- The `modelSpecs` docs explicitly say that having a spec list disables `modelSelect`, `parameters`, and `presets` unless those are re-enabled in the `interface` object.
- The `modelSpecs.prioritize` docs say that when set to `true`, a model spec is always selected in the UI and this may prevent users from selecting different endpoints for the selected spec.
- The `modelSpecs.group` docs say that if the group matches an endpoint name, the spec appears nested under that endpoint in the selector menu.

In practice, this means our first goal should be to preserve the normal `Camer AI` endpoint model picker and keep the `Agents` and `Assistants` sections configured as their own product areas.

That is why the experimental overlay uses:

- `interface.modelSelect: true`
- `interface.parameters: true`
- `interface.presets: true`
- `modelSpecs.enforce: false`
- `modelSpecs.prioritize: false`
- `group: "Camer AI"` for every MoE model spec

This is the best documented configuration path we found to try to expose both surfaces at once.

Important:

This is still experimental. The LibreChat docs describe the individual behaviors, but they do not explicitly guarantee how custom endpoint models and model specs merge in the selector for a custom endpoint named `Camer AI`. The grouping strategy is an inference from the documented `group` behavior and from the note that custom endpoint names must match exactly in presets.

## Prerequisites

Apply the required secrets first:

```bash
kubectl apply -f librechat-credentials-env.example.yaml -n librechat
kubectl apply -f librechat-api-key-secret.example.yaml -n librechat
```

Create real secret manifests from those examples before applying them. Do not commit production secrets to Git.

## Basic Deployment

If you want plain LibreChat without MoE presets:

```bash
helm install mongodb bitnami/mongodb \
  --set auth.enabled=false \
  --set persistence.enabled=false \
  -n default
```

```bash
helm upgrade --install librechat oci://ghcr.io/danny-avila/librechat-chart/librechat \
  --set mongodb.enabled=false \
  --values values.yaml \
  -n default
```

## Experimental Deployment With Camer AI + MoE

If you want to test both the normal `Camer AI` model list and the MoE specs together:

```bash
helm upgrade --install librechat oci://ghcr.io/danny-avila/librechat-chart/librechat \
  --set mongodb.enabled=false \
  --values values.yaml \
  --values values-camer-ai-moe.experimental.yaml \
  -n default
```

This overlay replaces the embedded `configYamlContent`, so keep the base file first and the experimental overlay second.

## Notes

- `values.yaml` is the source of truth for the simplest deployment path.
- `values-camer-ai-moe.experimental.yaml` is the best documented attempt to expose both `Camer AI` and MoE in one UI, but it still needs validation in your running LibreChat instance.
- The chart values in this repo assume a custom endpoint and a separate MongoDB release.
- If you change the namespace, update your secret manifests and Helm commands to match.
