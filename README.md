## Overview

This repo contains a minimal LibreChat Helm deployment plus an optional Mixture of Experts (MoE) overlay for teams that want specialist agent presets on top of a normal chat deployment.

Use:

- `values.yaml` for the base deployment
- `values-moe.yaml` only if you want the optional MoE model presets

The base deployment already includes:

- MongoDB connection and Meilisearch integration
- A custom OpenAI-compatible endpoint named `Camer AI`
- File upload limits for assistants
- Basic agent and assistant runtime settings
- Title generation and summarization settings compatible with the configured models

The MoE overlay adds:

- `modelSpecs` entries for an orchestrator and specialists
- Specialist routing presets mapped to the models already exposed by `Camer AI`

## Files

- [values.yaml](/home/virtualhost/librechat/values.yaml): base LibreChat deployment
- [values-moe.yaml](/home/virtualhost/librechat/values-moe.yaml): optional MoE overlay
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

### Optional MoE

The MoE layer is intentionally separate so a basic deployment stays easy to understand.

It adds these presets:

- `moe-orchestrator`
- `moe-strategy`
- `moe-creative`
- `moe-analysis`
- `moe-vision`
- `moe-fast`

Each preset uses one of the models already available in the `Camer AI` endpoint, so there is no second provider to configure.

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

## Deployment With MoE

If you want the optional MoE specialists, layer the overlay file on top of the base values:

```bash
helm upgrade --install librechat oci://ghcr.io/danny-avila/librechat-chart/librechat \
  --set mongodb.enabled=false \
  --values values.yaml \
  --values values-moe.yaml \
  -n default
```

Because `values-moe.yaml` overrides `librechat.configYamlContent`, keep the base file first and the MoE overlay second.

## Notes

- `values.yaml` is the source of truth for the simplest deployment path.
- `values-moe.yaml` is optional and should only be used when you want specialist presets in the UI.
- The chart values in this repo assume a custom endpoint and a separate MongoDB release.
- If you change the namespace, update your secret manifests and Helm commands to match.
