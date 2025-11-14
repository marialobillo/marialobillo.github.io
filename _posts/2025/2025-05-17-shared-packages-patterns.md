---
layout: post
title:  "Shared Packages in Monorepos: Patterns"
date:   2025-10-30 10:07:00 +0100
categories: monorepos microservices packages shared  
permalink: /shared-packages-patterns
---

# Shared Packages Best Practices: Lessons from Production

## The Full Journey

In this series, we've covered:

- **[Part 1](link):** The nightmare of code duplication across microservices
- **[Part 2](link):** How to build shared packages in a monorepo

Today, I'm sharing the hard-earned lessons from 6 months of using shared packages in production. These are the rules that saved us from disaster, and the mistakes we made so you don't have to.

---

## The Golden Rule of Shared Packages

After 6 months and countless discussions, our team converged on one principle:

> **"Extract infrastructure, not business logic."**

Let me explain why this matters.

### ✅ Good Candidates for Shared Packages

**1. Infrastructure Code** (technology wrappers)
````typescript
// @radarkit/logger - Logging wrapper
// @radarkit/kafka-client - Event bus wrapper
// @radarkit/database - Prisma utilities
// @radarkit/redis - Cache wrapper
// @radarkit/monitoring - Metrics and tracing
````

**Why:** These are **how** you do things. Every service logs, publishes events, and queries databases. One implementation ensures consistency.

**2. Type Definitions** (contracts between services)
````typescript
// @radarkit/types
export interface DomainEvent { ... }
export interface User { ... }
export interface Source { ... }
````

**Why:** Types define contracts. When services communicate, they must speak the same language.

**3. Cross-cutting Utilities** (used everywhere)
````typescript
// @radarkit/validation
export const isValidUrl = (url: string): boolean => { ... }
export const isValidEmail = (email: string): boolean => { ... }
````

**Why:** Pure functions with no business logic. URL validation is URL validation, regardless of domain.

### ❌ Bad Candidates (Keep These Local)

**1. Business Logic** (domain-specific)
````typescript
// ❌ DON'T extract
// @radarkit/subscription-calculator
export const calculatePrice = (user: User, plan: Plan): number => {
  // Complex business rules specific to your domain
}
````

**Why:** Business logic changes frequently and is domain-specific. Sharing it creates tight coupling.

**2. Service-Specific Implementation**
````typescript
// ❌ DON'T extract
// @radarkit/auth-strategies
export class JwtStrategy { ... }
export class OAuthStrategy { ... }
````

**Why:** Only auth-service needs this. Extracting creates unnecessary complexity.

**3. Experimental Code**
````typescript
// ❌ DON'T extract (yet)
// New ML-based ranking algorithm
// Wait until stable and used by 2+ services
````

**Why:** Premature extraction creates maintenance burden. Wait until patterns emerge.

---

## The Rule of Three

This is my #1 decision framework:
````
Code used in 1 place → Keep it local
Code used in 2 places → Start discussing extraction
Code used in 3+ places → MUST extract to package
````

### Real Example from RadarKit

**Month 1:** Only auth-service needs JWT validation
````typescript
// auth-service/src/auth/jwt.validator.ts
export const validateJwt = (token: string) => { ... }
````
**Decision: Keep local** ✅

**Month 2:** sources-service also needs JWT validation (protecting APIs)
````typescript
// sources-service/src/middleware/auth.middleware.ts
export const validateJwt = (token: string) => { ... }  // Copy-pasted
````
**Decision: Start discussion** ⚠️

**Month 3:** scraper-service needs it too (authenticated scraping)
````typescript
// scraper-service/src/auth/validator.ts
export const validateJwt = (token: string) => { ... }  // Third copy!
````
**Decision: EXTRACT NOW** 🚨
````typescript
// packages/auth-utils/src/jwt.ts
export const validateJwt = (token: string) => { ... }
````

---

## Versioning Strategy: The Non-Negotiable Rules

Semantic versioning isn't optional. It's what prevents 3 AM production incidents.

### Semantic Versioning 101
````
MAJOR.MINOR.PATCH
  |     |     |
  |     |     └─ Bug fixes (backward compatible)
  |     └─────── New features (backward compatible)
  └───────────── Breaking changes
````

### Real-World Examples

#### PATCH (1.0.0 → 1.0.1)
````typescript
// Before (buggy)
export const createLogger = (name: string) => {
  return winston.createLogger({
    level: 'info',  // 🐛 Hardcoded, ignores env var
  });
}

// After (fixed)
export const createLogger = (name: string) => {
  return winston.createLogger({
    level: process.env.LOG_LEVEL || 'info',  // ✅ Fixed
  });
}
````

**Impact:** All services can upgrade immediately. No code changes needed.

#### MINOR (1.0.1 → 1.1.0)
````typescript
// Before
export const createLogger = (name: string) => { ... }

// After - ADD new optional parameter
export const createLogger = (
  name: string, 
  options?: { level?: string }  // ✅ New, but optional
) => { ... }
````

**Impact:** Existing code still works. New feature available if needed.
````typescript
// Old code - still works
const logger = createLogger('auth-service');

// New code - uses new feature
const logger = createLogger('auth-service', { level: 'debug' });
````

#### MAJOR (1.1.0 → 2.0.0)
````typescript
// Before (1.x)
export const createLogger = (name: string) => { ... }

// After (2.0.0) - BREAKING: required parameter
export const createLogger = (
  name: string, 
  environment: string  // ⚠️ Now REQUIRED
) => { ... }
````

**Impact:** All services must update their code:
````typescript
// Before
const logger = createLogger('auth-service');  // ❌ Breaks with 2.0.0

// After
const logger = createLogger('auth-service', process.env.NODE_ENV);  // ✅
````

### Migration Strategy for Breaking Changes

When we released `@radarkit/logger@2.0.0`:

**Step 1: Announce the change**
````markdown
## @radarkit/logger 2.0.0 - BREAKING CHANGES

### What Changed
- `createLogger()` now requires `environment` parameter
- Removed deprecated `Logger` class

### Migration Guide
```typescript
// Before (1.x)
const logger = createLogger('service-name');

// After (2.0.0)
const logger = createLogger('service-name', process.env.NODE_ENV);
```

### Why This Change
- Better environment-specific logging
- Clearer configuration

### Timeline
- April 1: v2.0.0 released
- April 1-15: Both v1.x and v2.x supported
- April 15: v1.x deprecated
- May 1: v1.x support ends
````

**Step 2: Gradual migration**
````json
// Some services still on 1.x
"@radarkit/logger": "^1.2.0"

// Others adopt 2.x when ready
"@radarkit/logger": "^2.0.0"
````

**Step 3: Support period**
- v1.x receives critical bug fixes for 1 month
- Team has time to migrate without pressure
- No forced upgrade

---

## Package Organization Patterns

### Pattern 1: Single-Purpose Packages (Recommended)
````
packages/
├── logger/              ← Just logging
├── kafka-client/        ← Just Kafka
├── redis-client/        ← Just Redis
├── monitoring/          ← Just metrics/tracing
└── validation/          ← Just validation utils
````

**Pros:**
- ✅ Clear responsibility
- ✅ Easy to understand
- ✅ Version independently
- ✅ Small bundle size

### Pattern 2: Domain Packages (Sometimes)
````
packages/
├── infrastructure/      ← All infrastructure (logger, kafka, redis)
├── types/              ← All types
└── utils/              ← All utilities
````

**Pros:**
- ✅ Fewer packages to manage
- ✅ Easier for small teams

**Cons:**
- ❌ Large dependencies
- ❌ Version changes affect everyone
- ❌ Unclear responsibility

**My recommendation:** Start with Pattern 1. It scales better.

### Pattern 3: Layered Packages (Advanced)
````
packages/
├── core/               ← Base utilities (no dependencies)
│   ├── types/
│   └── constants/
│
├── infrastructure/     ← Depends on core
│   ├── logger/        (uses @radarkit/types)
│   ├── kafka/         (uses @radarkit/types)
│   └── database/      (uses @radarkit/types)
│
└── domain/            ← Depends on infrastructure
    ├── auth/          (uses @radarkit/logger)
    └── validation/    (uses @radarkit/logger)
````

**Dependency flow:**
````
core → infrastructure → domain → services
````

**When to use:** Large organizations with 20+ services.

---

## Testing Strategies

### Test Packages in Isolation
````typescript
// packages/logger/__tests__/logger.test.ts
import { createLogger } from '../src';

describe('Logger', () => {
  beforeEach(() => {
    // Reset environment
    delete process.env.LOG_LEVEL;
  });

  it('should use default log level', () => {
    const logger = createLogger('test');
    expect(logger.level).toBe('info');
  });

  it('should respect LOG_LEVEL env var', () => {
    process.env.LOG_LEVEL = 'debug';
    const logger = createLogger('test');
    expect(logger.level).toBe('debug');
  });

  it('should include service name in metadata', () => {
    const logger = createLogger('my-service');
    expect(logger.defaultMeta.service).toBe('my-service');
  });

  it('should handle errors gracefully', () => {
    const logger = createLogger('test');
    expect(() => {
      logger.error('Test error', new Error('Something failed'));
    }).not.toThrow();
  });
});
````

### Integration Tests in Services
````typescript
// services/auth-service/__tests__/integration/logging.test.ts
import { createLogger } from '@radarkit/logger';

describe('Logging Integration', () => {
  it('should log with correct service name', () => {
    const logger = createLogger('auth-service');
    
    // Capture console output
    const consoleSpy = jest.spyOn(console, 'log');
    
    logger.info('Test message', { userId: '123' });
    
    expect(consoleSpy).toHaveBeenCalledWith(
      expect.stringContaining('auth-service')
    );
    expect(consoleSpy).toHaveBeenCalledWith(
      expect.stringContaining('userId')
    );
  });
});
````

### E2E Tests Across Services
````typescript
// __tests__/e2e/event-flow.test.ts
describe('Event Flow E2E', () => {
  it('should publish and consume events with correct types', async () => {
    // Start services
    const authService = await startService('auth');
    const sourcesService = await startService('sources');
    
    // Publish event from auth
    await authService.createUser({ email: 'test@example.com' });
    
    // Verify sources received it
    await eventually(() => {
      const user = sourcesService.findUser('test@example.com');
      expect(user).toBeDefined();
    });
  });
});
````

---

## Documentation: Make Packages Easy to Use

### Every Package Needs a README
````markdown
# @radarkit/logger

Structured logging for RadarKit microservices.

## Installation
```bash
npm install @radarkit/logger
```

## Quick Start
```typescript
import { createLogger } from '@radarkit/logger';

const logger = createLogger('my-service');

logger.info('User logged in', { userId: '123' });
logger.error('Operation failed', { error: err.message });
```

## API

### createLogger(serviceName, options?)

Creates a Winston logger instance.

**Parameters:**
- `serviceName` (string): Name of your service
- `options.level` (string, optional): Log level. Default: `process.env.LOG_LEVEL || 'info'`
- `options.environment` (string, optional): Environment. Default: `process.env.NODE_ENV`

**Returns:** Winston Logger instance

**Example:**
```typescript
const logger = createLogger('auth-service', {
  level: 'debug',
  environment: 'development'
});
```

## Environment Variables

- `LOG_LEVEL`: Set log level (`debug`, `info`, `warn`, `error`)
- `NODE_ENV`: Environment (`development`, `production`)

## Log Format

**Development:**
````
10:23:45 [auth-service] info: User logged in
{
  "userId": "123"
}