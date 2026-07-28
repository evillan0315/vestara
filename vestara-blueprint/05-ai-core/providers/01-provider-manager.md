---
title: "Provider Manager — AI Provider Routing & Management"
volume: "05-ai-core"
book: "Book 3: AI Architecture"
version: "1.0.0"
status: "approved"
owner: "@ai-engineer"
last-reviewed: "2025-07-23"
next-review: "2026-01-23"
tags: ["providers", "routing", "open-code", "ollama", "openai"]
---

# Provider Manager
## AI Provider Routing, Fallback, and Cost Management

> **The Provider Manager is the central routing layer that abstracts all AI providers behind a unified interface. Users choose the best model for each task; the Provider Manager handles the rest.**

---

## 🎯 PROVIDER INTERFACE

```typescript
interface AIProvider {
  readonly id: ProviderId;               // 'opencode' | 'ollama' | 'openai' | 'anthropic' | 'google'
  readonly name: string;                  // Human-readable name
  readonly models: AIModel[];             // Available models list
  readonly features: ProviderFeatures;    // Capability bitmask
  
  // Core
  complete(request: CompletionRequest): Promise<CompletionResponse>;
  
  // Streaming
  stream(request: CompletionRequest): AsyncIterable<StreamChunk>;
  
  // Utility
  countTokens(text: string): number;
  estimateCost(input: string, output: string): CostEstimate;
  healthCheck(): Promise<HealthStatus>;
}
```

---

## 🔄 PROVIDER PRIORITY CHAIN

Default priority order (user-configurable via Settings):

```
1. opencode/*     → Free, no API key, cloud
2. ollama/*       → Local, privacy-first, on-demand
3. openai/*       → Paid, cloud, highest quality
4. anthropic/*    → Paid, cloud, long context
5. google/*       → Paid, cloud, multimodal
```

---

## 📦 PROVIDER STATES

| State | Meaning | User Visible |
|-------|---------|--------------|
| `available` | Ready to serve | ✅ Active in model picker |
| `degraded` | High latency or rate limited | ⚠️ Warning badge |
| `unavailable` | Missing API key or service down | ❌ Grayed out |
| `loading` | Starting (Ollama on-demand boot) | 🔄 Spinner with progress |
| `error` | Failed health check | ❌ Red badge |

---

## 🛠️ PROVIDER IMPLEMENTATIONS

### OpenCode (Default Provider)
- **Type**: Free cloud provider
- **Auth**: No API key required
- **Mode**: Embedded via iframe on port 4096
- **Models**: `opencode/deepseek-v4-flash-free`, `opencode/mimo-v2.5-free`, etc.
- **Limits**: Rate-limited, free tier
- **Status**: Always available

### Ollama (Local Provider)
- **Type**: Local on-premise
- **Auth**: None (local only)
- **Startup**: On-demand (no daemon)
- **Models**: `ollama/deepseek-coder`, `ollama/llama2`, user-installed
- **Limits**: Hardware-dependent
- **Status**: On-demand loading

### OpenAI, Anthropic, Google (External Providers)
- **Type**: Paid cloud
- **Auth**: API keys in `.env`
- **Models**: Full model catalog per provider
- **Limits**: API key rate limits, cost-based
- **Status**: Available when configured

---

## 📊 FALLBACK STRATEGY

```typescript
async function executeWithFallback(request: CompletionRequest): Promise<CompletionResponse> {
  const chain = getProviderChain(request.preferences);
  
  for (const provider of chain) {
    try {
      return await provider.complete(request);
    } catch (error) {
      logProviderError(provider.id, error);
      if (isRetryable(error) && retryCount < MAX_RETRIES) {
        await delay(EXPONENTIAL_BACKOFF(retryCount));
        continue;
      }
    }
  }
  
  throw new VestaraError('ALL_PROVIDERS_FAILED', 'All AI providers are currently unavailable');
}
```

---

## 🔗 CROSS-REFERENCES
- `05-ai-core/providers/` — Full provider documentation
- `services/agents/src/providers/` — Provider implementations
- `04-platform/PLATFORM_OVERVIEW.md` — Platform integration

---

**END OF PROVIDER MANAGER SPEC**

*The Provider Manager makes all AI providers interchangeable — users choose the best model, not the platform.*
