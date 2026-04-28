## Overview

This repo contains a LibreChat Helm configuration with a custom `Camer AI` endpoint, Agents support, MCP integration, and MoE model specs in a single deployable file.

Use:

- `values.yaml` as the single source of truth for deployment

The deployment includes:

- MongoDB connection and Meilisearch integration
- A custom OpenAI-compatible endpoint named `Camer AI`
- Agents enabled in both environment and UI config
- MCP server configuration (including `coder-mcp` OAuth settings)
- File upload limits and upload rate limits
- MoE model specs mapped to LibreChat Agent presets (`agent_id`)

## Files

- [values.yaml](/home/virtualhost/librechat-playground/values.yaml): unified LibreChat deployment values
- [values-camer-ai-moe.experimental.yaml](/home/virtualhost/librechat-playground/values-camer-ai-moe.experimental.yaml): legacy experimental reference (no longer required for deployment)
- [librechat-credentials-env.example.yaml](/home/virtualhost/librechat-playground/librechat-credentials-env.example.yaml): example application secret manifest
- [librechat-api-key-secret.example.yaml](/home/virtualhost/librechat-playground/librechat-api-key-secret.example.yaml): example API key secret manifest

## What Is Configured

### Runtime and Endpoints

- `ENDPOINTS` includes `agents` (`openAI,azureOpenAI,google,anthropic,agents`)
- `Camer AI` is configured as a custom OpenAI-compatible endpoint at `https://api.ai.camer.digital/v1`
- `Camer AI` model catalog is explicitly defined in `configYamlContent`
- Title and summary model are set to `qwen3-8b`

### Agents and MoE

- Agents UI controls are enabled (`use/create/share/public`)
- Agent runtime config includes allowed provider `Camer AI`, capabilities, and recursion/citation limits
- `modelSpecs` includes:
  - `moe-orchestrator`
  - `moe-strategy`
  - `moe-creative`
  - `moe-analysis`
  - `moe-vision`
  - `moe-fast`
- Each MoE entry points to `endpoint: agents` with a configured `agent_id`

### MCP

- MCP UI controls are enabled (`use/create`)
- `mcpSettings.allowedDomains` includes `coder.coder.svc.cluster.local`
- `mcpServers.coder-mcp` is configured as `streamable-http` with OAuth endpoints

## Prerequisites

Apply the required secrets first:

```bash
kubectl create ns librechat
kubectl apply -f librechat-credentials-env.example.yaml -n librechat
kubectl apply -f librechat-api-key-secret.example.yaml -n librechat
```

Create real secret manifests from those examples before applying them. Do not commit production secrets to Git.

## Deployment

Install MongoDB separately (as configured by `MONGO_URI`):

```bash
helm install mongodb bitnami/mongodb \
  --set auth.enabled=false \
  --set persistence.enabled=false \
  -n librechat
```

Deploy LibreChat with the unified values file:

```bash
helm upgrade --install librechat oci://ghcr.io/danny-avila/librechat-chart/librechat \
  --set mongodb.enabled=false \
  --values values.yaml \
  -n librechat
```

## Notes

- `values.yaml` is the only required values file for this setup.
- `values-camer-ai-moe.experimental.yaml` remains as historical reference.
- If you change namespace, update secret manifests, URLs, and Helm commands consistently.
- If your in-cluster service DNS differs from this repo defaults, update `MONGO_URI` and MCP URLs accordingly.
