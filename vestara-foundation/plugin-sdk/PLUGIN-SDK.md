---
id: "FND-008"
title: "Plugin SDK — Extension Architecture and Lifecycle"
owner: "@developer-platform"
status: "draft"
blueprint-ref: "10-developer-platform/SDK.md"
foundation-version: "1.0.0"
---

# Plugin SDK Specification

## Plugins Are First-Class Citizens — Manifest, Capabilities, Permissions, UI, Lifecycle

> **Every extension to Vestara is a plugin. Plugins can provide tools, commands, UI panels, themes, providers, and workflows. The Plugin SDK ensures every plugin has the same structure, lifecycle, and security model.**

---

## Plugin Manifest

Every plugin has a `manifest.json`:

```json
{
  "id": "my-plugin",
  "name": "My Vestara Plugin",
  "version": "1.0.0",
  "description": "What this plugin does",
  "author": "Plugin Author",
  "license": "MIT",
  "icon": "./icon.svg",
  
  "vestara": {
    "minVersion": "1.0.0",
    "maxVersion": "2.0.0"
  },
  
  "capabilities": [
    "tools:register",
    "commands:register",
    "ui:panels",
    "events:subscribe",
    "settings:provide"
  ],
  
  "permissions": [
    "filesystem:read",
    "network:connect"
  ],
  
  "tools": [
    { "id": "my-plugin.my-tool", "path": "./tools/my-tool.js" }
  ],
  
  "commands": [
    { "id": "my-plugin.my-command", "title": "Run My Command" }
  ],
  
  "ui": {
    "panels": [
      { "id": "my-plugin.panel", "title": "My Panel", "path": "./ui/panel.js" }
    ],
    "settings": [
      { "id": "my-plugin.settings", "title": "My Settings", "path": "./ui/settings.js" }
    ],
    "themes": [
      { "id": "my-plugin.theme", "name": "My Theme", "path": "./ui/theme.css" }
    ]
  },
  
  "events": {
    "subscribes": ["project:created", "conversation:message.sent"],
    "publishes": ["my-plugin:custom-event"]
  },
  
  "lifecycle": {
    "activateOnStartup": false,
    "deactivateOnIdle": true
  }
}
```

---

## Plugin Interface

```typescript
/**
 * Every plugin implements this interface.
 */
interface VestaraPlugin {
  /** Plugin metadata from manifest */
  readonly manifest: PluginManifest;
  
  /** Plugin context (set by Plugin Loader) */
  context: PluginContext | null;

  /**
   * Called when the plugin is activated.
   * Register tools, commands, UI panels, event handlers here.
   */
  activate(context: PluginContext): Promise<void>;
  
  /**
   * Called when the plugin is deactivated.
   * Clean up resources, unregister handlers.
   */
  deactivate(): Promise<void>;
  
  /**
   * Handle an event the plugin subscribed to.
   */
  handleEvent?(event: VestaraEvent): Promise<void>;
}

interface PluginManifest {
  id: string;
  name: string;
  version: string;
  description: string;
  author: string;
  license: string;
  icon?: string;
  
  vestara: {
    minVersion: string;
    maxVersion?: string;
  };
  
  capabilities: PluginCapability[];
  permissions: Permission[];
  
  tools?: ToolRegistration[];
  commands?: CommandRegistration[];
  ui?: UIRegistration;
  events?: EventRegistration;
  
  lifecycle?: PluginLifecycle;
}

type PluginCapability =
  | 'tools:register'
  | 'commands:register'
  | 'ui:panels'
  | 'ui:settings'
  | 'ui:themes'
  | 'events:subscribe'
  | 'events:publish'
  | 'settings:provide'
  | 'provider:register';

type Permission =
  | 'filesystem:read'
  | 'filesystem:write'
  | 'network:connect'
  | 'network:serve'
  | 'shell:execute'
  | 'memory:read'
  | 'memory:write'
  | 'storage:local'
  | 'ui:notifications';
```

---

## Plugin Context

```typescript
/**
 * Context provided to every plugin on activation.
 */
interface PluginContext {
  /** Plugin's isolated storage directory */
  storagePath: string;
  
  /** Plugin's configuration (from settings) */
  config: Record<string, unknown>;
  
  /** Plugin logger (prefixed with plugin ID) */
  logger: Logger;
  
  // ─── Plugin API ─────────────────────────────────────────
  
  /** Register a tool */
  registerTool(tool: Tool): Promise<void>;
  
  /** Unregister a tool */
  unregisterTool(toolId: string): Promise<void>;
  
  /** Register a command */
  registerCommand(command: Command): Promise<void>;
  
  /** Register a UI panel */
  registerPanel(panel: PanelDefinition): Promise<void>;
  
  /** Register a settings page */
  registerSettingsPage(page: SettingsPageDefinition): Promise<void>;
  
  /** Subscribe to events */
  subscribe(eventPattern: string, handler: EventHandler): Unsubscribe;
  
  /** Publish an event */
  emit(eventType: string, payload: Record<string, unknown>): Promise<void>;
  
  /** Get service by capability */
  getService<T>(capability: string): T | null;
  
  /** Get current user context */
  getUserContext(): UserContext | null;
  
  /** Show notification */
  showNotification(notification: PluginNotification): Promise<void>;
}
```

---

## Plugin Lifecycle

```
Installed → Enabled → Active → Disabled → Uninstalled
                ↓                    ↑
           Activation Error    Deactivation Error
```

| Phase | Description |
|-------|-------------|
| **Installed** | Plugin files on disk, not yet verified |
| **Verifying** | Manifest validation, capability checks, permission review |
| **Enabled** | Passed verification, ready to activate |
| **Activating** | `activate()` called, registering capabilities |
| **Active** | Fully operational |
| **Disabling** | `deactivate()` called, cleanup |
| **Disabled** | Plugin inactive, files preserved |
| **Uninstalled** | Files removed, data cleaned |

---

## Plugin Permission Model

```typescript
/**
 * Permission request shown to user on first activation.
 * Users can grant, deny, or grant-with-restrictions.
 */
interface PermissionRequest {
  pluginId: string;
  pluginName: string;
  requestedPermissions: Permission[];
  description: string;                // Why each permission is needed
  canBePartial: boolean;              // Can user grant a subset?
}

/**
 * Permission state for each installed plugin.
 */
interface PluginPermissions {
  pluginId: string;
  granted: Permission[];
  denied: Permission[];
  restricted: Record<string, PermissionRestriction>;
}

interface PermissionRestriction {
  paths?: string[];                    // Filesystem scope
  hosts?: string[];                    // Network scope
  rateLimit?: number;                  // Calls per minute
}
```

---

## Plugin Security

| Threat | Mitigation |
|--------|------------|
| Malicious code execution | Sandboxed VM, no host access |
| Unauthorized file access | Capability-based permissions |
| Data exfiltration | Network permissions, rate limiting, audit |
| Privilege escalation | Manifest verification, signature validation |
| Supply chain | Signed plugins, hash verification |
| Resource abuse | Timeout, memory limits, CPU quotas |

---

## Plugin Distribution

```yaml
marketplace:
  distribution:
    - "Local install from file (.vestara-plugin)"
    - "Marketplace install (Gen 3)"
    - "Development mode (symlinked)"
  
  verification:
    - "Manifest schema validation"
    - "Permission review (automated)"
    - "Content hash verification"
    - "Signature verification (future)"
  
  updates:
    - "Version check on startup"
    - "Background update download"
    - "Update with user confirmation"
```

---

*The Plugin SDK ensures every extension to Vestara follows the same architecture. Plugins are not afterthoughts — they are first-class citizens with the same lifecycle, security, and capability model as built-in features.*
