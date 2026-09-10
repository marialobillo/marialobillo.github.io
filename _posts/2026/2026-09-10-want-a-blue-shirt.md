---
layout: post
title: "I want a blue shirt in size M, to the cart using Agent"
date: 2026-06-14
categories: [testing, tdd, python]
tags: [ai-agent, llm, python, cart, rag, tool-calling, fastapi, ai]
---

# From "I want a blue shirt in size M" to a real order line

Most checkout flows assume the user already knows exactly what they want in
the catalog's own terms: an exact size, an exact color, a SKU. But nobody
actually thinks that way. People write (or say) things like *"I want a blue
shirt in size M and a pair of jeans, size 32"* — natural language, with
ambiguity, no product IDs attached.

That gap between how people talk and how a catalog is modeled is a small but
real problem, and a good case study for tool calling: turning free text into
structured data a system can actually act on, without making anything up.

## The hard part isn't "just ask an LLM"

Asking a model to "extract the products from this text" and trusting it to
return something parseable is fragile — the output format can drift, it can
invent a product that isn't in your catalog, or be ambiguous about which
catalog line it actually means.

The more reliable approach is forcing the model to call a function with a
strict schema — tool calling with a forced `tool_choice`, not a suggestion.
The tool receives the real catalog as part of the context, and the schema
requires each extracted line to include:

- the original text fragment it came from (traceability)
- the quantity
- the `product_id` that matches the catalog — or an explicit `null` if there
  is no confident match, instead of the model inventing an ID that doesn't
  exist

That explicit `null` is the piece that matters most: it forces the model to
admit uncertainty instead of hallucinating a product. An order line with no
match can be surfaced to the user for manual confirmation, instead of
silently slipping something wrong into the cart.

## An architecture decision, not just a prompt

The order extractor is defined as an interface (`OrderExtractor`) in the
domain layer, with zero dependency on which LLM provider sits behind it.
Each provider (Anthropic, OpenRouter, or a deterministic "fake" extractor
for tests and cost-free demos) implements that same interface separately.

The practical result: swapping models means adding one new file, not
touching business logic. The cart, order confirmation, stock checks — none
of it knows or cares which model is extracting the lines. That separation is
what makes it trivial to try the system with a free model before deciding
whether a better one is worth paying for, instead of a rewrite.

## Try it

The repo runs out of the box with a deterministic extractor (no API key, no
cost), so anyone can clone it and see it work immediately. Switching to a
real model with actual tool calling is a single environment variable.

[https://github.com/marialobillo/shop-agent-demo]


