---
id: "FND-006"
title: "Universal Interface Specification — Common Contracts for All Runtime Components"
owner: "@chief-architect"
status: "ratified"
blueprint-ref: "14-engineering/ARCHITECTURE.md"
foundation-version: "1.0.0"
---

# Universal Interface Specification
## Every Runtime Component Implements These Contracts

> **Every service, runtime, agent, provider, and plugin in the Vestara platform implements the same core interfaces. This ensures consistent lifecycle management, observability, and replaceability across the entire platform.**

---

## 1. 🔷 VestaraService (Every Runtime Component)

```typescript
/**
 * Every runtime component in Vestara implements this interface.
 * From the Kernel to the simplest utility service.
 */
interface VestaraService {
  /** Unique service identifier */
  readonly id: string;
  
  /** Semantic version of this service */
  readonly version: string;
  
  /** Current service status */
  readonly status: ServiceStatus;

  /**
   * Initialize the service with optional configuration.
   * Called during system boot, before start().
   * Should not start processing — just prepare resources.
   */
  initialize(config?: Record<string, unknown>): Promise<void>;

  /**
   * Start the service. Begin accepting requests, processing events.
   * Called after all dependencies are initialized.
   */
  start(): Promise<void>;

  /**
   * Gracefully stop the service. Drain in-flight work.
   * Called during system shutdown or service replacement.
   */
  stop(): Promise<void>;

  /**
   * Return current health status including dependent services.
   */
  health(): Promise<HealthStatus>;

  /**
   * Dispose of all resources. Called after stop().
   * The service should not be used after dispose().
   */
  dispose(): Promise<void>;
}

type ServiceStatus = 
  | 'uninitialized'    // Not yet initialized
  | 'initializing'     // initialize() in progress
  | 'initialized'      // Ready to start
  | 'starting'         // start() in progress
  | 'running'          // Operational
  | 'degraded'         // Running but with issues
  | 'stopping'         // stop() in progress
  | 'stopped'          // Stopped, can be disposed
  | 'disposed';        // Resources released

interface HealthStatus {
  status: 'healthy' | 'degraded' | 'unhealthy';
  serviceId: string;
  version: string;
  uptime: number;                  // Seconds since start
  lastHealthCheck: string;         // ISO 8601
  dependencies: HealthDependency[];
  metrics?: Record<string, number>;
  message?: string;                // Human-readable status
}

interface HealthDependency {
  id: string;
  status: 'healthy' | 'degraded' | 'unhealthy' | 'unknown';
  latency: number;                 // ms
  lastChecked: string;             // ISO 8601
}
```

---

## 2. 📦 Repository (Data Access)

```typescript
/**
 * Base repository pattern for data access.
 * Every entity type has a repository implementation.
 */
interface Repository<T extends Entity> {
  findById(id: string): Promise<T | null>;
  findAll(filter?: Filter<T>): Promise<T[]>;
  create(input: CreateInput<T>): Promise<T>;
  update(id: string, input: UpdateInput<T>): Promise<T>;
  delete(id: string): Promise<void>;
  count(filter?: Filter<T>): Promise<number>;
}

interface Entity {
  id: string;
  createdAt: string;    // ISO 8601
  updatedAt: string;    // ISO 8601
}

type Filter<T> = Partial<Record<keyof T, unknown>> & {
  search?: string;
  page?: number;
  limit?: number;
  sortBy?: keyof T;
  sortOrder?: 'asc' | 'desc';
};
```

---

## 3. 📡 EventBus (Pub/Sub)

```typescript
/**
 * Typed event bus for inter-service communication.
 */
interface EventBus {
  /** Emit an event to all subscribers */
  emit(event: VestaraEvent): Promise<void>;
  
  /** Subscribe to events matching a pattern */
  subscribe<T = unknown>(
    pattern: string,           // Exact match or wildcard: 'project:*'
    handler: EventHandler<T>,
    options?: SubscribeOptions
  ): Unsubscribe;
  
  /** Subscribe to exactly one event, then auto-unsubscribe */
  once<T = unknown>(
    type: string,
    handler: EventHandler<T>
  ): Unsubscribe;
  
  /** Remove all handlers for a pattern */
  unsubscribeAll(pattern?: string): void;
  
  /** Get event bus metrics */
  getMetrics(): EventBusMetrics;
}

interface VestaraEvent {
  id: string;                    // UUID v7
  type: string;                  // 'domain:action'
  version: number;               // Schema version
  timestamp: string;             // ISO 8601
  source: string;                // Service ID
  actor?: { id: string; role: 'user' | 'system' | 'agent' };
  payload: Record<string, unknown>;
  metadata: {
    correlationId: string;
    causationId?: string;
    retryCount: number;
    ttl: number;                 // Seconds
  };
}

type EventHandler<T = unknown> = (event: VestaraEvent & { payload: T }) => Promise<void>;

interface SubscribeOptions {
  /** Maximum number of concurrent handlers */
  concurrency?: number;
  
  /** Handler timeout in ms */
  timeout?: number;
  
  /** Retry on failure */
  retry?: boolean;
  
  /** Maximum retry attempts */
  maxRetries?: number;
}

type Unsubscribe = () => void;

interface EventBusMetrics {
  totalEmitted: number;
  totalProcessed: number;
  totalFailed: number;
  avgLatency: number;            // ms
  activeSubscribers: number;
  handlerCounts: Record<string, number>;
}
```

---

## 4. 🔒 Logger (Observability)

```typescript
/**
 * Structured logging interface.
 * Every service logs through this interface.
 */
interface Logger {
  debug(message: string, context?: LogContext): void;
  info(message: string, context?: LogContext): void;
  warn(message: string, context?: LogContext): void;
  error(message: string, context?: LogContext & { error?: Error }): void;
  
  /** Create a child logger with additional context */
  child(context: LogContext): Logger;
  
  /** Flush pending log entries */
  flush(): Promise<void>;
}

interface LogContext {
  serviceId?: string;
  requestId?: string;
  correlationId?: string;
  userId?: string;
  [key: string]: unknown;
}

interface LogEntry {
  timestamp: string;             // ISO 8601
  level: 'debug' | 'info' | 'warn' | 'error';
  message: string;
  context: LogContext;
  error?: {
    name: string;
    message: string;
    stack?: string;
  };
}
```

---

## 5. 📊 Metrics (Observability)

```typescript
/**
 * Metrics collection interface.
 */
interface MetricsCollector {
  /** Increment a counter */
  increment(name: string, value?: number, tags?: Record<string, string>): void;
  
  /** Record a histogram value */
  record(name: string, value: number, tags?: Record<string, string>): void;
  
  /** Set a gauge value */
  gauge(name: string, value: number, tags?: Record<string, string>): void;
  
  /** Time a function execution */
  time<T>(name: string, fn: () => Promise<T>, tags?: Record<string, string>): Promise<T>;
  
  /** Create a child collector with default tags */
  child(defaultTags: Record<string, string>): MetricsCollector;
}

interface MetricDefinition {
  name: string;
  type: 'counter' | 'histogram' | 'gauge';
  description: string;
  unit?: string;
  tags?: string[];
}
```

---

## 6. ⚙️ Configuration

```typescript
/**
 * Configuration provider interface.
 */
interface ConfigurationProvider {
  /** Get a typed configuration value */
  get<T>(key: string, defaultValue?: T): T;
  
  /** Set a configuration value at runtime */
  set<T>(key: string, value: T): void;
  
  /** Check if configuration key exists */
  has(key: string): boolean;
  
  /** Get all configuration keys */
  keys(): string[];
  
  /** Load configuration from source */
  load(): Promise<void>;
  
  /** Subscribe to configuration changes */
  onChange(keys: string[], handler: ConfigChangeHandler): Unsubscribe;
}

type ConfigChangeHandler = (changes: Record<string, unknown>) => Promise<void>;
```

---

## 7. 🔐 Auth (Authentication & Authorization)

```typescript
/**
 * Authentication and authorization interface.
 */
interface AuthService extends VestaraService {
  /** Authenticate user credentials */
  authenticate(credentials: Credentials): Promise<Session>;
  
  /** Verify JWT and return user context */
  verifyToken(token: string): Promise<UserContext>;
  
  /** Refresh an expiring token */
  refreshToken(token: string): Promise<{ token: string; expiresAt: string }>;
  
  /** Revoke a session */
  revokeSession(sessionId: string): Promise<void>;
  
  /** Check if user has required role/permission */
  checkPermission(userId: string, permission: string, resource?: string): Promise<boolean>;
  
  /** Get all permissions for a user */
  getPermissions(userId: string): Promise<string[]>;
}

interface UserContext {
  userId: string;
  username: string;
  role: 'admin' | 'editor' | 'user';
  permissions: string[];
  sessionId: string;
  expiresAt: string;
}

interface Credentials {
  username: string;
  password: string;
  otp?: string;              // Future: MFA
}

interface Session {
  token: string;
  user: UserContext;
  expiresIn: number;          // seconds
}
```

---

## 8. 🔄 State Machine (Common Pattern)

```typescript
/**
 * Generic state machine interface.
 * Used by StateMachine, Agent, Mission, Workflow, etc.
 */
interface StateMachine<S extends string, E extends string> {
  /** Current state */
  readonly currentState: S;
  
  /** Available transitions from current state */
  readonly availableTransitions: E[];
  
  /** Send an event to transition state */
  send(event: E, payload?: unknown): Promise<void>;
  
  /** Check if a transition is valid */
  canTransition(event: E): boolean;
  
  /** Get the full state history */
  getHistory(): StateTransition<S, E>[];
  
  /** Reset to initial state */
  reset(): Promise<void>;
  
  /** Get current state metadata */
  getStateInfo(): StateInfo<S>;
}

interface StateTransition<S, E> {
  from: S;
  to: S;
  event: E;
  timestamp: string;           // ISO 8601
  payload?: unknown;
}

interface StateInfo<S> {
  state: S;
  enteredAt: string;           // ISO 8601
  duration: number;            // ms in current state
  metadata: Record<string, unknown>;
}
```

---

## 9. 🔗 Service Registry

```typescript
/**
 * Service registry for service discovery.
 */
interface ServiceRegistry {
  /** Register a service with its capabilities */
  register(service: VestaraService, capabilities: string[]): Promise<void>;
  
  /** Unregister a service */
  unregister(serviceId: string): Promise<void>;
  
  /** Find a service by capability */
  findByCapability(capability: string): VestaraService | null;
  
  /** Find all services matching a capability */
  findAllByCapability(capability: string): VestaraService[];
  
  /** List all registered services */
  listServices(): ServiceInfo[];
  
  /** Watch for service changes */
  watch(callback: (event: ServiceRegistryEvent) => void): Unsubscribe;
}

interface ServiceInfo {
  id: string;
  version: string;
  status: ServiceStatus;
  capabilities: string[];
  dependencies: string[];
  uptime: number;
}

interface ServiceRegistryEvent {
  type: 'registered' | 'unregistered' | 'status-changed';
  serviceId: string;
  timestamp: string;
}
```

---

## 10. 📝 Serialization

```typescript
/**
 * Serialization interface for data transfer.
 */
interface Serializer<T, R = string> {
  serialize(input: T): R;
  deserialize(input: R): T;
  validate(input: unknown): input is T;
}
```

---

## Implementation Requirements

| Interface | Required For | Gen |
|-----------|-------------|-----|
| `VestaraService` | ALL runtime components | 1 |
| `Repository<T>` | ALL data access layers | 1 |
| `EventBus` | Foundation | 1 |
| `Logger` | ALL services | 1 |
| `MetricsCollector` | ALL services | 1 |
| `ConfigurationProvider` | Foundation | 1 |
| `AuthService` | Identity service | 1 |
| `StateMachine<S,E>` | Agents, Missions, Workflows | 1 |
| `ServiceRegistry` | Service Bus | 1 |
| `Serializer<T,R>` | Plugin boundaries | 2 |

---

**The Universal Interface ensures that every component in Vestara shares the same lifecycle, communicates through the same patterns, and can be composed into larger systems without adaptation layers.**
