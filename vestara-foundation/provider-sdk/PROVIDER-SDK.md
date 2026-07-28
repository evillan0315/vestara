---
id: "FND-007"
title: "Provider SDK — AI Provider Abstraction Contract"
owner: "@ai-engineer"
status: "ratified"
blueprint-ref: "05-ai-core/providers/01-provider-manager.md"
foundation-version: "1.0.0"
---

# Provider SDK Specification
## Any AI Provider Plugs In Through This Contract

> **No OpenAI-specific code. No Anthropic-specific code. Every provider implements the same contract: models, chat, streaming, embeddings, vision, speech, tools, files. Adding a new provider means implementing this SDK — not patching the core.**

---

## Provider Contract

```typescript
/**
 * Every AI provider in Vestara implements this interface.
 * OpenCode, Ollama, OpenAI, Anthropic, Google — all equal.
 */
interface AIProvider extends VestaraService {
  /** Provider identifier (e.g., 'opencode', 'ollama', 'openai') */
  readonly id: ProviderId;
  
  /** Human-readable provider name */
  readonly name: string;
  
  /** Available models */
  readonly models: AIModel[];
  
  /** Provider capabilities */
  readonly capabilities: ProviderCapabilities;

  // ─── Core AI ────────────────────────────────────────────
  
  /** Complete a chat request (non-streaming) */
  complete(request: CompletionRequest): Promise<CompletionResponse>;
  
  /** Stream a chat response */
  stream(request: CompletionRequest): AsyncIterable<StreamChunk>;
  
  // ─── Utilities ──────────────────────────────────────────
  
  /** Count tokens in text */
  countTokens(text: string): TokenCount;
  
  /** Estimate cost for a request */
  estimateCost(request: CostRequest): CostEstimate;
  
  /** Check provider health */
  healthCheck(): Promise<ProviderHealth>;
  
  // ─── Model Management ───────────────────────────────────
  
  /** List available models */
  listModels(): Promise<AIModel[]>;
  
  // ─── Gen 2+ Capabilities ────────────────────────────────
  
  /** Generate embeddings (Gen 2) */
  embed?(request: EmbeddingRequest): Promise<EmbeddingResponse>;
  
  /** Analyze image (Gen 3) */
  analyzeImage?(request: ImageAnalysisRequest): Promise<ImageAnalysisResponse>;
  
  /** Generate image (Gen 3) */
  generateImage?(request: ImageGenerationRequest): Promise<ImageGenerationResponse>;
  
  /** Transcribe speech to text (Gen 3) */
  transcribe?(request: TranscriptionRequest): Promise<TranscriptionResponse>;
  
  /** Generate speech from text (Gen 3) */
  synthesize?(request: SynthesisRequest): Promise<SynthesisResponse>;
}

type ProviderId = 'opencode' | 'ollama' | 'openai' | 'anthropic' | 'google' | string;
```

---

## Data Types

```typescript
// ─── Models ────────────────────────────────────────────────

interface AIModel {
  id: string;                    // 'gpt-4o', 'claude-3.5-sonnet'
  provider: ProviderId;          // 'openai'
  name: string;                  // 'GPT-4o'
  description?: string;
  
  // Capabilities
  capabilities: ModelCapabilities;
  
  // Context
  contextWindow: number;         // Max tokens
  maxOutput: number;             // Max output tokens
  
  // Performance
  pricing: ModelPricing;
  avgLatencyMs?: number;         // Estimated
    
  // Status
  status: 'available' | 'degraded' | 'unavailable';
}

interface ModelCapabilities {
  chat: boolean;
  streaming: boolean;
  functionCalling: boolean;
  parallelToolCalls: boolean;
  vision: boolean;               // Gen 3
  embeddings: boolean;           // Gen 2
  speech: boolean;               // Gen 3
  imageGeneration: boolean;      // Gen 3
  jsonMode: boolean;             // Structured output
}

interface ModelPricing {
  inputPerMillionTokens: number;   // USD
  outputPerMillionTokens: number;  // USD
  cachedInputPerMillionTokens?: number;
}

interface ProviderCapabilities {
  maxConcurrentRequests: number;
  rateLimit: { requestsPerMinute: number; tokensPerMinute: number };
  features: ModelCapabilities;
}

// ─── Requests ──────────────────────────────────────────────

interface CompletionRequest {
  model: string;
  messages: Message[];
  system?: string;
  temperature?: number;          // 0-2, default 0.7
  maxTokens?: number;
  stream?: boolean;
  tools?: ToolDefinition[];
  toolChoice?: 'auto' | 'required' | 'none';
  stop?: string[];
  topP?: number;
  frequencyPenalty?: number;
  presencePenalty?: number;
  seed?: number;
  user?: string;                 // End-user identifier for monitoring
}

interface Message {
  role: 'system' | 'user' | 'assistant' | 'tool';
  content: string | ContentBlock[];
  name?: string;
  tool_calls?: ToolCall[];
  tool_call_id?: string;
}

type ContentBlock = TextBlock | ImageBlock | ToolResultBlock;

interface TextBlock { type: 'text'; text: string; }
interface ImageBlock { type: 'image'; source: { type: 'base64' | 'url'; data: string; media_type?: string; }; }

interface ToolDefinition {
  type: 'function';
  function: {
    name: string;
    description: string;
    parameters: Record<string, unknown>;  // JSON Schema
  };
}

// ─── Responses ───────────────────────────────────────────────

interface CompletionResponse {
  id: string;
  model: string;
  provider: ProviderId;
  choices: Choice[];
  usage: TokenUsage;
  cost: number;
  latency: number;               // Total ms
  created: string;               // ISO 8601
}

interface Choice {
  index: number;
  message: Message;
  finishReason: 'stop' | 'length' | 'tool_calls' | 'content_filter' | 'error';
}

interface TokenUsage {
  prompt: number;
  completion: number;
  total: number;
  cached?: number;
}

// ─── Streaming ───────────────────────────────────────────────

interface StreamChunk {
  type: 'token' | 'tool-call' | 'tool-result' | 'error' | 'done' | 'meta';
  data: StreamToken | ToolCall | ToolResult | StreamError | StreamMeta;
  index: number;
}

interface StreamToken {
  content: string;
  model: string;
}

interface StreamMeta {
  usage: TokenUsage;
  cost: number;
  latency: number;
}

interface StreamError {
  code: string;
  message: string;
  retryable: boolean;
}

// ─── Tool Calling ────────────────────────────────────────────

interface ToolCall {
  id: string;
  type: 'function';
  function: {
    name: string;
    arguments: string;           // JSON string
  };
}

interface ToolResult {
  tool_call_id: string;
  content: string;
  status: 'success' | 'error';
}

// ─── Embeddings (Gen 2) ─────────────────────────────────────

interface EmbeddingRequest {
  model: string;
  input: string | string[];
  dimensions?: number;
}

interface EmbeddingResponse {
  model: string;
  embeddings: number[][];
  usage: TokenUsage;
}

// ─── Utilities ───────────────────────────────────────────────

interface TokenCount {
  count: number;
  model: string;
}

interface CostRequest {
  model: string;
  inputTokens: number;
  outputTokens?: number;
}

interface CostEstimate {
  inputCost: number;
  outputCost: number;
  totalCost: number;
  currency: 'USD';
}

interface ProviderHealth {
  status: 'healthy' | 'degraded' | 'unhealthy';
  latency: number;               // ms
  lastChecked: string;
  models: AIModel[];
  message?: string;
}
```

---

## Provider Configuration Schema

```typescript
interface ProviderConfig {
  id: ProviderId;
  enabled: boolean;
  priority: number;              // Lower = preferred
  models: string[];              // Whitelisted models (empty = all)
  
  // Authentication
  apiKey?: string;               // Injected from OS keychain, never logged
  baseUrl?: string;              // Custom endpoint
  organization?: string;         // OpenAI org
  
  // Limits
  maxConcurrentRequests: number;
  rateLimit: { requestsPerMinute: number; tokensPerMinute: number };
  maxRetries: number;
  
  // Cost management
  monthlyBudget?: number;
  costAlertThreshold?: number;
  
  // Fallback
  fallbackProviders: ProviderId[];
}
```

---

## Example: OpenAI Provider Implementation

```typescript
class OpenAIProvider implements AIProvider {
  readonly id = 'openai' as const;
  readonly name = 'OpenAI';
  models: AIModel[] = [];
  
  async initialize(config: ProviderConfig): Promise<void> {
    // Load API key from OS keychain
    // Discover models from API
    // Register with Provider Runtime
  }
  
  async complete(request: CompletionRequest): Promise<CompletionResponse> {
    // Transform Vestara request → OpenAI format
    // Call OpenAI API
    // Transform response → Vestara format
    // Track costs, latency, usage
  }
  
  async *stream(request: CompletionRequest): AsyncIterable<StreamChunk> {
    // Transform request → OpenAI format
    // Stream from OpenAI API
    // Transform chunks → StreamChunk format
    // Handle errors, emit meta when done
  }
  
  async healthCheck(): Promise<ProviderHealth> {
    // Call OpenAI models endpoint
    // Measure latency
    // Return status
  }
}
```

---

## Provider State Machine

```
Registered → Available → Degraded → Unavailable
    ↑            |            |           |
    └────────────┴────────────┴───────────┘
          (Health check recovery)
```

| State | Meaning | User Visible |
|-------|---------|--------------|
| `registered` | Configured but not yet verified | ⏳ Verifying |
| `available` | Healthy, accepting requests | ✅ Available |
| `degraded` | High latency, rate limited | ⚠️ Slow |
| `unavailable` | API key missing, service down | ❌ Unavailable |

---

## Provider Fallback Chain

```
1. Try primary provider (user's default model)
2. If fails (rate limit, timeout, error):
   a. Log error with provider
   b. Check if retryable → retry with backoff (3x)
   c. If not retryable → try next provider in chain
3. If all providers fail → return error to user
```

---

*The Provider SDK ensures no provider-specific code exists in the Vestara core. Every provider is a plugin that implements this contract.*
