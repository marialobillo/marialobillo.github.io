---
layout: post
title: "Making Architectural Decisions Under Real Constraints: Building a Scalable Intelligence Platform"
date: 2025-09-27 20:00:00 +0100
tags: [architecture, nodejs, nestjs, ddd, decision-making]
---

When you're starting a new backend project, one of the first questions you face is: **monolith or microservices?** It's tempting to reach for the latest patterns, but the right answer depends entirely on your constraints.

Recently, I had to make this decision for a competitive intelligence platform that monitors signals from competitors—pricing changes, product releases, job postings, API updates, and more. Here's how we navigated the decision, what we chose, and why.

## The Project Context

The platform needs to:
- Monitor 9+ different types of signals from various sources (web scraping, APIs, feeds)
- Process them asynchronously and send notifications
- Scale incrementally as we add more signal types
- Support a small team initially (2-3 devs), growing to 5+ soon

**The challenge:** We needed to ship features quickly while building something maintainable and scalable. The team would grow, and we couldn't afford to accumulate tech debt that would slow us down in 6 months.

## The Architectural Question

Starting fresh gives you options—too many options, sometimes. We had three paths:

1. **Microservices from day one**
2. **Simple monolithic API** (Express + PostgreSQL)
3. **Modular monolith** with clear boundaries

Each had merit. None was obviously "correct."

## Option 1: Microservices from Day One

**The case for it:**
- Each signal type (pricing, releases, jobs, etc.) could be its own service
- Independent scaling: CPU-heavy scraping services separate from I/O-bound APIs
- Clear team ownership as we grow
- Modern, "the way it's done"

**Why we didn't choose it:**
- **Operational overhead is real.** You need service discovery, API gateways, distributed tracing, separate deployments. With a 2-person team initially, this eats development time.
- **Premature boundaries.** We're still learning the domain. Microservices force you to commit to boundaries early—get them wrong and you're doing cross-service calls constantly.
- **Overkill for MVP.** We need to validate product-market fit first, not prove we can orchestrate 15 services.

**The lesson here:** Microservices solve problems you don't have yet. When you have them, you'll know.

## Option 2: Simple Express Monolith

**The case for it:**
- **Ship fast.** No architectural overhead, just start coding.
- Minimal infrastructure: one server, one database.
- Easy debugging: everything's in one place.

**Why we didn't choose it:**
- **Boundaries erode fast.** Without structure, you end up with tangled dependencies. "Just one quick import" turns into spaghetti.
- **Hard to extract later.** When you do need to scale a specific module, you're doing surgery on a codebase with no clear seams.
- **Team coordination becomes painful.** As the team grows, everyone's touching the same files.

**The lesson:** Fast now, slow later. We'd be refactoring in 6 months when we should be shipping features.

## What We Chose: Modular Monolith with DDD-Lite

We went with a **modular monolith**—a single deployable application, but organized into isolated modules (bounded contexts in DDD terms) that communicate through events.

**Why this made sense:**

### 1. Single Deployment, Clear Boundaries
We get the operational simplicity of a monolith (one deploy, one server, one log stream) with the organizational benefits of microservices (modules own their domain, communicate via contracts).

### 2. Event-Driven from the Start
Even though all modules are in the same process, they communicate through Kafka events. When the `monitoring` module detects a pricing change, it publishes a `PricingChangedEvent`. The `notifications` module subscribes and sends alerts.

**Why Kafka in a monolith?** Two reasons:
- **Decoupling:** Modules don't call each other directly. This keeps boundaries clean and makes extraction trivial later.
- **Async by default:** Long-running tasks (scraping, notifications) don't block request handlers.

### 3. Extractable by Design
When we eventually need to scale the scraping module independently (it's CPU-bound), we can extract it to a separate service with zero changes to the event contracts. The rest of the system won't even notice.

### 4. Team-Friendly
As the team grows, devs can own entire bounded contexts without stepping on each other's toes. Clear module boundaries = fewer merge conflicts, clearer PRs.

## The Structure

The codebase is organized around bounded contexts—each representing a distinct area of the domain:

- **Monitoring:** Detects competitor signals (pricing, releases, jobs, etc.)
- **Notifications:** Handles alerting via email, Slack, webhooks
- **Competitors:** Manages competitor data and sources


Each context follows a layered architecture: domain logic, application use cases, infrastructure adapters. Modules communicate exclusively through domain events published to Kafka—no direct function calls across boundaries.

This structure means we can reason about each module independently. When you're working on the pricing detector, you don't need to understand how notifications work. You just publish a `PricingChangedEvent` and move on.

## Implementation Decisions

Beyond the high-level architecture, a few specific choices shaped how we work:

**Shared domain patterns.** We built base classes for common patterns—`DomainEvent` for all events, `Entity` for domain objects with identity, `Result` for explicit error handling. This enforces consistency: every module speaks the same language.

Initially, this felt like over-engineering. But it's paid off: code reviews are easier, new team members onboard faster, and refactoring is safer because contracts are explicit.

**Event-first communication.** Modules publish domain events to Kafka rather than calling each other's functions. This felt heavyweight at first—"why not just call the notification service?"—but it's made the codebase more resilient. If the notifications module is slow or crashes, it doesn't block signal detection. Events queue up and get processed when the service recovers.

**Local development with Docker.** We use Docker Compose so new developers can run `docker-compose up` and have PostgreSQL and Kafka running locally. No "works on my machine" issues, no manual installation steps.


## Trade-offs We Accepted

Every decision has costs. Here's what we're living with:

**1. More boilerplate than raw Express**  
NestJS is opinionated. You write more files (module, controller, service, repository) than you would with Express routes. But this structure pays off as the codebase grows.

**2. Kafka adds infrastructure complexity**  
Running Kafka locally is heavier than "just use function calls." But the decoupling and async-first design are worth it.

**3. Discipline required**  
Maintaining module boundaries takes effort. Without code reviews that enforce "no cross-module imports," the architecture degrades. We need to stay vigilant.

**4. Not optimized for heavy CPU workloads yet**  
If scraping becomes a bottleneck, we'll need to extract it. But we'll cross that bridge when we have the data to justify it.

## Reflection: Would I Make the Same Decision Again?

**Yes, given the same constraints.**

If we had:
- A 10-person team from day one → Maybe microservices
- Simple CRUD with no async processing → Maybe simpler monolith
- A team experienced with distributed systems → Maybe microservices

But with a small team, rapid feature development needs, and domain we're still learning, **modular monolith was the pragmatic choice.**

## What Surprised Me

**The value of shared domain patterns.** I initially thought `DomainEvent`, `Entity`, `Result` were over-engineering. But they've enforced consistency across all modules and made code reviews easier. Everyone speaks the same language.

**How easy Docker Compose makes onboarding.** Seeing new devs run `npm run docker:up` and have a working environment in 2 minutes is worth the upfront setup.

## What I'd Change

**Start with fewer signals.** We planned for 9 signals from the start. In hindsight, implementing 2-3 end-to-end first would've validated the architecture faster and reduced scope.

**More upfront schema design.** We're iterating on database schemas as we go. Some upfront design sessions would've saved refactor time.

## Key Lesson

**Architecture decisions should serve your constraints, not textbook ideals.**

There's no "best" architecture—only trade-offs. Microservices aren't inherently better than monoliths. DDD isn't always the answer. The right choice depends on:
- Team size and experience
- Time to market pressure
- Domain complexity
- Operational maturity

Start with the simplest thing that could work, but build in seams so you can evolve. A well-structured monolith beats a poorly executed microservices architecture every time.

---

**Building something similar? I'd love to hear how you approached the architecture decision.** Connect with me on [LinkedIn](https://www.linkedin.com/in/maria-lobillo/).

---

*This is part of a series on building scalable backend systems. Next up: implementing bounded contexts with NestJS and handling domain events with Kafka.*