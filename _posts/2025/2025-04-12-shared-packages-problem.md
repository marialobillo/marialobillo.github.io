---
layout: post
title:  "Shared Packages in Monorepos: The Problem"
date:   2025-10-28 10:06:00 +0100
categories: monorepos microservices packages shared  
permalink: /shared-packages-problem
---

# The Hidden Cost of Copy-Paste in Microservices

## The 3 AM Production Bug

It's 3:42 AM. My phone buzzes with a PagerDuty alert:

> **CRITICAL: Kafka message delivery failure - 87% error rate**

I grab my laptop, still half-asleep, and SSH into our production cluster. After 15 minutes of investigation, I find the culprit: a bug in our Kafka event publishing logic. A simple off-by-one error in the retry mechanism.

"Easy fix," I think. I write the patch in 5 minutes.

Then reality hits me.

This bug exists in **five different microservices**. Each service has its own copy of the Kafka client code, copy-pasted months ago when we were moving fast and "breaking things."

My "5-minute fix" now requires:
- Opening 5 separate pull requests
- Waiting for 5 different code reviews (different team members own each service)
- Running 5 separate CI pipelines
- Deploying 5 services independently
- Monitoring 5 rollouts

**Total time: 2.5 days.**

**Time to actually fix the bug: 5 minutes.**

This is the hidden tax of code duplication in microservices. And we're all paying it.

---

## How We Got Here

Six months ago, our team started building RadarKit, a distributed content monitoring platform. We were excited about microservices:

- Independent deployment ✅
- Technology flexibility ✅
- Team autonomy ✅
- Fault isolation ✅

We split our monolith into services:
```
radarkit/
├── auth-service/
├── sources-service/
├── scraper-service/
├── alerts-service/
└── notification-service/
```

Everything was great. Until it wasn't.

---

## The Copy-Paste Disease

It started innocently. Our first two services needed to log events. Simple, right?
```typescript
// auth-service/src/utils/logger.ts
export class Logger {
  log(level: string, message: string, meta?: any) {
    console.log(JSON.stringify({
      timestamp: new Date().toISOString(),
      level,
      message,
      service: 'auth-service',
      ...meta
    }));
  }
}
```

Works perfectly. Then `sources-service` needed logging. Rather than setting up a shared package (which seemed like "premature optimization"), we just... copied the file.
```typescript
// sources-service/src/utils/logger.ts
export class Logger {
  log(level: string, message: string, meta?: any) {
    console.log(JSON.stringify({
      timestamp: new Date().toISOString(),
      level,
      message,
      service: 'sources-service',  // Just changed the service name
      ...meta
    }));
  }
}
```

Two weeks later, we needed Kafka integration. Same pattern:
```typescript
// auth-service/src/kafka/client.ts
export class KafkaClient {
  async publish(topic: string, message: any) {
    // 150 lines of Kafka connection, retry logic, error handling
  }
}

// sources-service/src/kafka/client.ts
export class KafkaClient {
  async publish(topic: string, message: any) {
    // Same 150 lines, copy-pasted
  }
}

// scraper-service/src/kafka/client.ts
export class KafkaClient {
  async publish(topic: string, message: any) {
    // You get the idea...
  }
}
```

By month three, we had:
- **3 copies** of logging code
- **5 copies** of Kafka client
- **4 copies** of validation utilities
- **5 copies** of database helpers
- **Countless copies** of shared types

---

## When Reality Caught Up

### Problem 1: The Version Drift

Three months in, I noticed something odd. Each service was logging differently:
```typescript
// auth-service logs:
{"timestamp":"2025-01-15T10:00:00Z","level":"info","message":"User logged in"}

// sources-service logs:
{"time":"2025-01-15T10:00:00Z","severity":"INFO","msg":"Source created"}

// scraper-service logs:
{"@timestamp":"2025-01-15T10:00:00Z","log.level":"info","event.original":"Scraping started"}
```

Why? Because each team had "improved" their copy independently. We now had three different logging formats, making centralized log analysis impossible.

### Problem 2: The Bug Multiplication

Remember that 3 AM bug? Here's what the original code looked like:
```typescript
// Buggy retry logic (in 5 services)
async publishWithRetry(topic: string, message: any) {
  const maxRetries = 3;
  let attempt = 0;
  
  while (attempt < maxRetries) {  // 🐛 Bug: should be <=
    try {
      await this.kafka.send({ topic, messages: [message] });
      return;
    } catch (error) {
      attempt++;
      await this.sleep(1000 * attempt);
    }
  }
  
  throw new Error('Max retries exceeded');
}
```

The bug was subtle: `attempt < maxRetries` meant we only tried 2 times, not 3. 

When I discovered this in `auth-service`, I had to search for the same pattern across all services:
```bash
$ git grep -n "attempt < maxRetries" services/
services/auth-service/src/kafka/client.ts:45:  while (attempt < maxRetries) {
services/sources-service/src/kafka/client.ts:52:  while (attempt < maxRetries) {
services/scraper-service/src/kafka/publisher.ts:38:  while (attempt < maxRetries) {
services/alerts-service/src/events/publisher.ts:41:  while (attempt < maxRetries) {
services/notification-service/src/kafka/producer.ts:29:  while (attempt < maxRetries) {
```

Five files. Five PRs. Five reviews. Five deploys.

### Problem 3: The Feature Inconsistency

We needed to add distributed tracing. The conversation in Slack:

> **Me:** "Adding trace IDs to Kafka events for distributed tracing"
> 
> **Sarah (Backend Dev):** "Oh, I already did that in sources-service last week!"
> 
> **Me:** "...it's not in auth-service"
> 
> **Tom (Backend Dev):** "I added it to scraper-service yesterday, different implementation though"

We had three different tracing implementations, none compatible with each other. Our tracing tool showed broken traces because services used different correlation ID formats.

### Problem 4: The Onboarding Nightmare

New developer joins the team:

> **New Dev:** "Where's the standard way to publish Kafka events?"
> 
> **Me:** "Well... check auth-service, but actually sources-service has a better implementation, oh wait, scraper-service has the latest retry logic..."
> 
> **New Dev:** "So... there's no standard?"
> 
> **Me:** "...not exactly."

---

## The Real Cost

Let's put numbers on this:

### Time Waste

Over 3 months:
- **47 hours** spent on duplicating fixes across services
- **23 hours** debugging inconsistencies between service implementations
- **18 hours** in meetings discussing "which implementation is the source of truth"
- **12 hours** onboarding new developers on "the patterns" (which weren't actually consistent)

**Total: 100 hours = 2.5 weeks of engineering time**

### Bug Multiplication

When we audited our codebase:
- **3 critical bugs** existed in multiple services
- **12 minor bugs** had been fixed in some services but not others
- **5 security patches** needed to be applied 5 times

### Technical Debt
```bash
$ cloc services/*/src/kafka/
# Kafka client implementation
Auth Service:     312 lines
Sources Service:  298 lines
Scraper Service:  334 lines
Alerts Service:   301 lines
Notification:     289 lines
-----------------------------------
Total:          1,534 lines

# 80% similar code = ~1,200 lines of pure duplication
```

**1,200 lines of code that should have been 300 lines in one place.**

---

## The Copy-Paste Cascade

The worst part? It gets worse over time:
```
Month 1: "Let's just copy this logger, it's only 50 lines"
           ↓
Month 2: "Let's just copy the Kafka client, it's already copied"
           ↓
Month 3: "Let's just copy the database helpers, everything else is copied"
           ↓
Month 4: "Let's just copy..." 
```

It becomes the norm. Each copy makes the next copy feel justified. The technical debt compounds.

---

## The Breaking Point

The final straw came during a security audit. We discovered a critical vulnerability in our JWT token validation:
```typescript
// Vulnerable code (in 3 services)
export const verifyToken = (token: string) => {
  return jwt.verify(token, process.env.JWT_SECRET);
  // 🚨 No algorithm verification - allows "none" algorithm attack
}
```

The fix was simple:
```typescript
export const verifyToken = (token: string) => {
  return jwt.verify(token, process.env.JWT_SECRET, {
    algorithms: ['HS256']  // Explicitly specify allowed algorithms
  });
}
```

But we had to:
1. Fix it in auth-service
2. Fix it in sources-service  
3. Fix it in alerts-service
4. Deploy all three within a tight window
5. Hope we didn't miss it in another service

**A one-line security fix became a multi-day, high-stakes deployment.**

That's when we knew we had to change something fundamental.

---

## The Hidden Questions

After this incident, I started asking myself:

- Why are we copy-pasting code in microservices when we spent years learning DRY in monoliths?
- How do companies like Google, Netflix, and Uber manage thousands of microservices without this problem?
- Is there a way to share code without losing the benefits of microservices?

The answer surprised me. **Yes, there is a better way.**

Companies like Google don't have this problem because they use **shared packages**. Not shared libraries that couple services together. Not going back to a monolith. But properly designed, versioned, shared packages in a monorepo.

---

## What's Coming Next

In the next post, I'll show you:

- How we restructured RadarKit to use shared packages
- The exact setup that eliminated 80% of our code duplication
- How we now deploy fixes to all services in minutes, not days
- The monorepo pattern that keeps microservices independent while sharing code

We went from:
- **5 PRs per bug fix** → **1 PR**
- **2-3 days per infrastructure change** → **30 minutes**
- **Inconsistent behavior** → **Guaranteed consistency**
- **Onboarding confusion** → **Clear patterns**

All while keeping our microservices **completely independent** in production.

---

## Your Turn

Have you experienced the copy-paste cascade in your microservices? How many copies of your logging code exist right now?

Count them:
```bash
# Try this in your codebase
git grep -l "class Logger" services/
```

Drop the number in the comments. I bet it's more than you think.

**Next post:** "The Solution - How to Build Shared Packages That Don't Suck"

---

*This is Part 1 of a 3-part series on building maintainable microservices. I'm María, Senior Backend Developer building RadarKit, a distributed content monitoring platform. Follow along as we solve real problems with real solutions.*

**Series:**
- Part 1: The Problem (you are here)
- Part 2: The Solution - Coming next week
- Part 3: Best Practices - Coming soon

---

*Found this helpful? Follow me on [Twitter/LinkedIn] for more posts on distributed systems, microservices, and lessons learned building production systems.*