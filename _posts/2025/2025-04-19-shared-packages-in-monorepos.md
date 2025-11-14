---
layout: post
title:  "Shared Packages in Monorepos: Implementation"
date:   2025-04-19 20:00:00 +0100
categories: monorepos microservices packages shared  
permalink: /shared-packages-implementation
---

# Shared Packages: The Solution to Microservices Code Duplication

## Quick Recap

In [Part 1](link-to-part-1), I shared how code duplication nearly destroyed our microservices architecture:

- A 5-minute bug fix took 2.5 days across 5 services
- 1,200 lines of duplicated code
- Inconsistent logging, tracing, and error handling
- Security vulnerabilities multiplied across services

Today, I'll show you exactly how we fixed it.

---

## The Lightbulb Moment

After the JWT vulnerability incident, I started researching how big tech companies handle this. I found something interesting:

**Google:** Single monorepo with 1000s of microservices, sharing code through internal packages

**Netflix:** Shared libraries for cross-cutting concerns, deployed independently

**Uber:** Common packages for infrastructure code (logging, metrics, tracing)

The pattern was clear: **Share infrastructure code, keep business logic independent.**

But there was confusion in our team:

> "Wait, aren't we going back to a monolith?"

No. Here's the crucial distinction:
```
MONOREPO = Code organization (one Git repository)
MICROSERVICES = Runtime architecture (independent processes)
SHARED PACKAGES = Code reuse strategy (DRY infrastructure code)
```

You can have all three together. In fact, that's the sweet spot.

---

## The Architecture Shift

### Before: Copy-Paste Hell
```
auth-service/           ← Separate repo
├── src/
│   ├── kafka/client.ts       (312 lines)
│   ├── logger.ts             (87 lines)
│   └── validation.ts         (124 lines)

sources-service/        ← Separate repo
├── src/
│   ├── kafka/client.ts       (298 lines) - DUPLICATED
│   ├── logger.ts             (91 lines)  - DUPLICATED
│   └── validation.ts         (119 lines) - DUPLICATED

scraper-service/        ← Separate repo
├── src/
│   ├── kafka/client.ts       (334 lines) - DUPLICATED
│   ├── logger.ts             (95 lines)  - DUPLICATED
│   └── validation.ts         (127 lines) - DUPLICATED
```

**Total infrastructure code: 1,587 lines (mostly duplicated)**

### After: Shared Packages
```
radarkit/                      ← One repo, many services
├── packages/                  ← Shared code
│   ├── logger/               (95 lines)  ← ONE implementation
│   ├── kafka-client/         (312 lines) ← ONE implementation
│   └── types/                (180 lines) ← ONE source of truth
│
└── services/                  ← Your microservices
    ├── auth-service/         (business logic only)
    ├── sources-service/      (business logic only)
    └── scraper-service/      (business logic only)
```

**Total infrastructure code: 587 lines**

**Reduction: 63% less code to maintain**

---

## Implementation: Step by Step

Let me show you exactly how we did this. Follow along and you can implement this today.

### Step 1: Create the Monorepo
```bash
# Create the structure
mkdir radarkit && cd radarkit
npm init -y

# Create folders
mkdir -p packages services infrastructure
```

### Step 2: Configure npm Workspaces
```json
// radarkit/package.json
{
  "name": "radarkit",
  "version": "1.0.0",
  "private": true,
  "workspaces": [
    "packages/*",
    "services/*"
  ],
  "scripts": {
    "build": "npm run build --workspaces",
    "dev": "npm run dev --workspaces",
    "test": "npm run test --workspaces"
  },
  "devDependencies": {
    "turbo": "^2.0.0",
    "typescript": "^5.3.0"
  }
}
```

**What this does:** npm creates symlinks between packages. When a service imports `@radarkit/logger`, it resolves to `packages/logger` locally.

### Step 3: Add Turborepo (optional but recommended)

Turborepo makes builds fast by caching and parallelization.
```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "dev": {
      "dependsOn": ["^build"],
      "cache": false,
      "persistent": true
    },
    "test": {
      "dependsOn": ["build"]
    }
  }
}
```

**Key insight:** `"dependsOn": ["^build"]` means "build dependencies first". So `logger` builds before `auth-service` that uses it.

---

## Building Your First Shared Package

Let's extract the logger that was duplicated across 5 services.

### Step 1: Create the Package Structure
```bash
cd packages
mkdir logger && cd logger
npm init -y
```

### Step 2: Package Configuration
```json
// packages/logger/package.json
{
  "name": "@radarkit/logger",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "default": "./dist/index.js"
    }
  },
  "files": ["dist"],
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "test": "jest"
  },
  "dependencies": {
    "winston": "^3.11.0"
  },
  "devDependencies": {
    "@types/node": "^20.10.0",
    "typescript": "^5.3.0",
    "jest": "^29.7.0"
  }
}
```

**Important fields:**
- `main`: Entry point for Node.js
- `types`: TypeScript definitions
- `exports`: Modern Node.js exports
- `files`: What to include when publishing

### Step 3: TypeScript Configuration
```json
// packages/logger/tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "types": ["node"]
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "__tests__"]
}
```

### Step 4: Implementation
```typescript
// packages/logger/src/index.ts
import winston from 'winston';

export interface LoggerConfig {
  serviceName: string;
  level?: string;
  environment?: string;
}

export const createLogger = (config: LoggerConfig | string) => {
  // Support both: createLogger('service-name') and createLogger({ ... })
  const options: LoggerConfig = typeof config === 'string' 
    ? { serviceName: config }
    : config;

  const {
    serviceName,
    level = process.env.LOG_LEVEL || 'info',
    environment = process.env.NODE_ENV || 'development',
  } = options;

  // Structured format for production
  const jsonFormat = winston.format.combine(
    winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
    winston.format.errors({ stack: true }),
    winston.format.json()
  );

  // Human-readable format for development
  const consoleFormat = winston.format.combine(
    winston.format.colorize(),
    winston.format.timestamp({ format: 'HH:mm:ss' }),
    winston.format.printf(({ timestamp, level, message, service, ...meta }) => {
      const metaStr = Object.keys(meta).length 
        ? '\n' + JSON.stringify(meta, null, 2) 
        : '';
      return `${timestamp} [${service}] ${level}: ${message}${metaStr}`;
    })
  );

  return winston.createLogger({
    level,
    format: jsonFormat,
    defaultMeta: { 
      service: serviceName,
      environment,
      hostname: process.env.HOSTNAME,
    },
    transports: [
      // Console for all environments
      new winston.transports.Console({
        format: environment === 'production' ? jsonFormat : consoleFormat,
      }),
      
      // File logs in production
      ...(environment === 'production' ? [
        new winston.transports.File({ 
          filename: 'logs/error.log', 
          level: 'error',
          maxsize: 5242880, // 5MB
          maxFiles: 5,
        }),
        new winston.transports.File({ 
          filename: 'logs/combined.log',
          maxsize: 5242880,
          maxFiles: 5,
        }),
      ] : []),
    ],
  });
};

// Export types
export type Logger = winston.Logger;
```

**What makes this good:**
- ✅ Consistent format across all services
- ✅ Different output for dev vs production
- ✅ Structured logging (JSON) for log aggregation
- ✅ Automatic metadata (service name, environment)
- ✅ File rotation in production

### Step 5: Add Tests
```typescript
// packages/logger/__tests__/logger.test.ts
import { createLogger } from '../src';

describe('Logger', () => {
  it('should create logger with service name', () => {
    const logger = createLogger('test-service');
    expect(logger.defaultMeta.service).toBe('test-service');
  });

  it('should accept string or config object', () => {
    const logger1 = createLogger('service-1');
    const logger2 = createLogger({ serviceName: 'service-2' });
    
    expect(logger1.defaultMeta.service).toBe('service-1');
    expect(logger2.defaultMeta.service).toBe('service-2');
  });

  it('should use environment variables', () => {
    process.env.LOG_LEVEL = 'debug';
    const logger = createLogger('test');
    
    expect(logger.level).toBe('debug');
  });
});
```

### Step 6: Build the Package
```bash
npm run build
```

You should see:
```
packages/logger/
├── dist/
│   ├── index.js          ← Compiled JavaScript
│   ├── index.d.ts        ← TypeScript definitions
│   └── index.js.map      ← Source maps
```

---

## Using the Shared Package

Now comes the magic. Let's use it in auth-service.

### Step 1: Install from Workspace
```bash
cd ../../services/auth-service
```
```json
// services/auth-service/package.json
{
  "name": "auth-service",
  "version": "1.0.0",
  "dependencies": {
    "@radarkit/logger": "*",  // ← The * means "use workspace version"
    "@nestjs/common": "^10.0.0",
    // ... other deps
  }
}
```

### Step 2: Use in Your Code
```typescript
// services/auth-service/src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { createLogger } from '@radarkit/logger';  // ← Import from package

async function bootstrap() {
  const logger = createLogger('auth-service');
  
  const app = await NestFactory.create(AppModule, {
    logger: false, // Disable default logger
  });
  
  // Use our logger
  app.useLogger(logger);
  
  const port = process.env.PORT || 3001;
  await app.listen(port);
  
  logger.info(`Auth service running on port ${port}`);
}

bootstrap();
```
```typescript
// services/auth-service/src/auth/auth.service.ts
import { Injectable } from '@nestjs/common';
import { createLogger } from '@radarkit/logger';

@Injectable()
export class AuthService {
  private logger = createLogger('auth-service');

  async login(email: string, password: string) {
    this.logger.info('Login attempt', { email });
    
    try {
      // ... authentication logic
      this.logger.info('Login successful', { email, userId: user.id });
      return user;
    } catch (error) {
      this.logger.error('Login failed', { email, error: error.message });
      throw error;
    }
  }
}
```

### Step 3: Install Dependencies
```bash
# From root
npm install
```

npm workspaces automatically creates symlinks:
```
services/auth-service/node_modules/@radarkit/logger 
  → symlink to packages/logger
```

### Step 4: Run It
```bash
npm run dev
```

Output:
```
10:45:23 [auth-service] info: Auth service running on port 3001
10:45:45 [auth-service] info: Login attempt
{
  "email": "user@example.com"
}
10:45:46 [auth-service] info: Login successful
{
  "email": "user@example.com",
  "userId": "uuid-here"
}
```

**Same logger. Every service. Every time.**

---

## Building More Shared Packages

Let's do two more critical packages: types and Kafka client.

### Package 2: Types (Type Safety Everywhere)
```bash
cd packages
mkdir types && cd types
npm init -y
```
```typescript
// packages/types/src/index.ts

// Domain entities
export interface User {
  id: string;
  email: string;
  name: string;
  role: 'user' | 'admin';
  createdAt: Date;
}

export interface Source {
  id: string;
  userId: string;
  url: string;
  type: 'rss' | 'web';
  status: 'active' | 'paused' | 'error';
  checkInterval: number;
  createdAt: Date;
}

export interface Alert {
  id: string;
  sourceId: string;
  userId: string;
  title: string;
  content: string;
  url: string;
  publishedAt: Date;
  createdAt: Date;
}

// Event types (for Kafka)
export interface DomainEvent<T = unknown> {
  type: string;
  timestamp: string;
  data: T;
  metadata: {
    service: string;
    version: string;
    correlationId: string;
    causationId?: string;
  };
}

// Specific event types
export type SourceCreatedEvent = DomainEvent<{
  sourceId: string;
  userId: string;
  url: string;
  type: 'rss' | 'web';
}>;

export type ContentFoundEvent = DomainEvent<{
  sourceId: string;
  title: string;
  content: string;
  url: string;
  publishedAt: string;
}>;

export type AlertCreatedEvent = DomainEvent<{
  alertId: string;
  userId: string;
  title: string;
}>;

// API types
export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
  };
}

export interface PaginatedResponse<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
  hasMore: boolean;
}
```

**Why this is powerful:**

Now when sources-service publishes an event and scraper-service consumes it, TypeScript **guarantees** they're using the same structure:
```typescript
// sources-service/src/sources/sources.service.ts
import { SourceCreatedEvent } from '@radarkit/types';

const event: SourceCreatedEvent = {
  type: 'source.created',
  timestamp: new Date().toISOString(),
  data: {
    sourceId: source.id,
    userId: source.userId,
    url: source.url,
    type: source.type,
  },
  metadata: {
    service: 'sources-service',
    version: '1.0.0',
    correlationId: uuid(),
  },
};

await kafka.publish('sources.created', event);
```
```typescript
// scraper-service/src/scraper/scraper.consumer.ts
import { SourceCreatedEvent } from '@radarkit/types';

async handleSourceCreated(message: SourceCreatedEvent) {
  // TypeScript knows the exact shape!
  const { sourceId, url, type } = message.data;
  
  // Type-safe access to all fields
  this.logger.info('New source to scrape', { sourceId, url });
  
  await this.startScraping(sourceId, url, type);
}
```

### Package 3: Kafka Client
```bash
cd packages
mkdir kafka-client && cd kafka-client
npm init -y
npm install kafkajs
npm install --save-dev @radarkit/types @radarkit/logger
```
```typescript
// packages/kafka-client/src/index.ts
import { Kafka, Producer, Consumer, EachMessagePayload } from 'kafkajs';
import { DomainEvent } from '@radarkit/types';
import { createLogger } from '@radarkit/logger';
import { v4 as uuid } from 'uuid';

export class KafkaClient {
  private kafka: Kafka;
  private producer: Producer;
  private logger = createLogger('kafka-client');
  
  constructor(
    private clientId: string,
    brokers: string[] = process.env.KAFKA_BROKERS?.split(',') || ['localhost:9092']
  ) {
    this.kafka = new Kafka({
      clientId,
      brokers,
      retry: {
        retries: 3,
        initialRetryTime: 300,
        factor: 2,
      },
    });
    
    this.producer = this.kafka.producer();
  }
  
  async connect(): Promise<void> {
    await this.producer.connect();
    this.logger.info('Kafka producer connected', { clientId: this.clientId });
  }
  
  async publish<T = unknown>(topic: string, event: DomainEvent<T>): Promise<void> {
    try {
      // Ensure correlation ID exists
      if (!event.metadata.correlationId) {
        event.metadata.correlationId = uuid();
      }
      
      await this.producer.send({
        topic,
        messages: [
          {
            key: event.metadata.correlationId,
            value: JSON.stringify(event),
            headers: {
              'event-type': event.type,
              'correlation-id': event.metadata.correlationId,
              'service': event.metadata.service,
            },
          },
        ],
      });
      
      this.logger.debug('Event published', { 
        topic, 
        type: event.type,
        correlationId: event.metadata.correlationId,
      });
    } catch (error) {
      this.logger.error('Failed to publish event', { 
        topic, 
        type: event.type,
        error: error.message,
      });
      throw error;
    }
  }
  
  createConsumer(groupId: string): Consumer {
    return this.kafka.consumer({ 
      groupId,
      sessionTimeout: 30000,
    });
  }
  
  async subscribe(
    consumer: Consumer,
    topic: string,
    handler: (event: DomainEvent) => Promise<void>
  ): Promise<void> {
    await consumer.connect();
    await consumer.subscribe({ topic, fromBeginning: false });
    
    await consumer.run({
      eachMessage: async ({ topic, partition, message }: EachMessagePayload) => {
        const correlationId = message.headers?.['correlation-id']?.toString();
        
        try {
          const event: DomainEvent = JSON.parse(message.value!.toString());
          
          this.logger.debug('Event received', { 
            topic, 
            type: event.type,
            correlationId,
          });
          
          await handler(event);
          
        } catch (error) {
          this.logger.error('Failed to process message', {
            topic,
            partition,
            offset: message.offset,
            correlationId,
            error: error.message,
          });
          throw error;
        }
      },
    });
  }
  
  async disconnect(): Promise<void> {
    await this.producer.disconnect();
    this.logger.info('Kafka producer disconnected');
  }
}
```

**What this gives you:**
- ✅ Consistent retry logic across all services
- ✅ Correlation IDs automatically handled
- ✅ Structured logging for debugging
- ✅ Type-safe event publishing/consuming
- ✅ Error handling in one place

---

## The Results

After migrating all services to use shared packages:

### Code Reduction
```
Before: 1,587 lines of duplicated infrastructure code
After:  587 lines in shared packages
Savings: 1,000 lines (63% reduction)
```

### Bug Fix Time
```
Before: 
- Find bug in one service
- Search for duplicates (git grep)
- Fix in 5 places
- 5 PRs, 5 reviews, 5 deploys
- Time: 2-3 days

After:
- Find bug in package
- Fix once
- 1 PR, 1 review
- All services get it on next rebuild
- Time: 30 minutes
```

### Feature Rollout
```
Example: Adding distributed tracing

Before:
- Implement in service 1: 2 hours
- Copy-paste to service 2-5: 4 hours
- Fix inconsistencies: 3 hours
- Total: 9 hours

After:
- Implement in @radarkit/kafka-client: 2 hours
- All services get it: automatic
- Total: 2 hours
```

### Developer Onboarding
```
Before:
New dev: "How do I log?"
You: "Check auth-service... actually sources has better impl... wait, use scraper's version"

After:
New dev: "How do I log?"
You: "import { createLogger } from '@radarkit/logger'"
```

### Build Pipeline
```bash
# One command builds everything in correct order
npm run build

# Turborepo output:
@radarkit/types: Build completed (1.2s)
@radarkit/logger: Build completed (0.8s)
@radarkit/kafka-client: Build completed (1.1s)
auth-service: Build completed (3.2s)
sources-service: Build completed (2.9s)
scraper-service: Build completed (4.1s)

Total time: 4.1s (parallel builds)
```

---

## Common Questions

### Q: "Isn't this coupling services together?"

**A: No.** Services are coupled at **build time** (good) but independent at **runtime** (critical).
```
Build time: 
  services depend on packages ✅

Runtime:
  auth-service → separate container
  sources-service → separate container
  scraper-service → separate container
  All deploy independently ✅
```

### Q: "What if I need to change a package?"

**A: Version it.**
```bash
# Make breaking change in @radarkit/logger
cd packages/logger
# Update to 2.0.0

# Services opt-in when ready
# auth-service uses logger@2.0.0
# sources-service still on logger@1.0.0
# No forced migration
```

### Q: "How do I test packages?"
```bash
# Test package in isolation
cd packages/logger
npm test

# Test service with package
cd services/auth-service
npm test  # Uses @radarkit/logger via symlink
```

---

## What's Next

We've now eliminated code duplication and built a solid foundation. But there are still questions:

- When should you extract code to a package?
- How do you version packages properly?
- What about breaking changes?
- How do you avoid the "god package" anti-pattern?

**Part 3** will cover best practices, patterns, and pitfalls to avoid.

---

## Try It Yourself

Want to implement this in your project? Here's the 30-minute starter:
```bash
# 1. Create monorepo
mkdir my-project && cd my-project
npm init -y

# 2. Add workspaces
echo '{
  "name": "my-project",
  "private": true,
  "workspaces": ["packages/*", "services/*"]
}' > package.json

# 3. Create your first package
mkdir -p packages/logger
cd packages/logger
npm init -y
# Add the logger code from above

# 4. Create a service that uses it
mkdir -p services/api
cd services/api
npm init -y
npm install @my-project/logger

# 5. Build and run
cd ../..
npm install
npm run build
```

---

## Your Turn

How much duplicated code exists in your microservices?
```bash
# Find your most duplicated file
find services/ -name "*.ts" -exec md5 {} \; | sort | uniq -d -w 32
```

Share your findings in the comments!

**Next up: Part 3 - Best Practices and Pitfalls**

---



**Series:**
- [Part 1: The Problem](link)
- Part 2: The Solution (you are here)
- Part 3: Best Practices - Coming next week

---
