# NovelVerse · Algorithms and implementation choices

These notes explain how the application uses established algorithms and its own product-specific rules. They do not claim that graph traversal or transaction patterns were invented for this app.

## 1. Validating an interactive story graph

**Problem:** a story can contain an unreachable scene, a cycle or a non-ending scene with no way forward.

The validator builds an adjacency map from enabled choices. A depth-first traversal maintains both a visited set and an active recursion set. Reaching an active node identifies a cycle. After traversal, nodes absent from the visited set are reported as unreachable.

It also checks for a valid start, destinations within the book, explicit endings, no outgoing choices from ending nodes, and one to three choices on non-ending nodes. For a proposed new edge, a reachability check from its destination back to its source detects whether that edge would close a cycle.

The traversal itself is `O(V + E)` in chapters and enabled choices, after the graph has been loaded. Structural reachability does not prove that every combination of story-state requirements is satisfiable; that is a different problem.

## 2. A small story-state rule evaluator

Choices carry declarative requirements and effects. Requirements recursively support `all`, `any` and `not`, then compare state values with operations such as equality, numeric bounds, membership and truthiness. Effects support scalar assignment and numeric addition.

For example, a choice can add 2 to `trust` and set `has_key = true`. A later option can require both `trust >= 3` and a truthy `has_key`. The server filters visible choices and rechecks the condition when a decision is submitted.

The useful part is the composition: authors can express route rules in data while the client receives the options available for the saved state.

## 3. Preventing a double advance

**Trigger:** a reader taps twice, retries after a slow connection, or chooses on two devices.

The decision path combines a unique request identifier with a transactional compare-and-swap update: advance only if the progress row is still on the expected source node. The successful transition and its ordered decision record are committed together. If an identical request was already applied, the handler can return that result; a competing different transition cannot advance the old node again.

The request ID handles repeated delivery. The expected-node condition handles competing choices. They solve related but distinct problems.

## 4. Charging once for a chapter with several scenes

Interactive scene nodes share a visible chapter number. Before spending tokens, the service finds the stable first node for that group. The unique user/chapter unlock and the spending record use that anchor.

Example: scenes A, B and C all belong to chapter 4. Unlocking A makes the chapter group accessible. Reaching B through another route does not turn B into a second purchase. Active subscription access is checked separately and does not spend tokens.

## 5. Duplicate-aware purchase and subscription records

The purchase service requests transaction information from Apple and checks the returned product identity. Balance increment and ledger insertion run in one database transaction. A unique purchase reference catches repeated credit attempts.

Webhook delivery uses a separate unique event record and processed timestamp. Consumable-purchase events do not overwrite subscription access merely because they have no subscription expiry. Cancellation and expiration are handled as different lifecycle events.

## 6. Text layout instead of a fixed character count

The reader normalizes paragraph breaks and common text entities, constructs attributed text with the selected typography, then uses TextKit containers sized to the reading area. Each container supplies the glyph and character range for one page.

This lets page boundaries respond to font, line spacing and available space. UIKit’s page-curl controller presents the resulting pages; SwiftUI coordinates the surrounding controls.

## 7. One session refresh for several waiting requests

The actor-based API client keeps an optional in-flight refresh task. The first request creates it; concurrent callers await that task. The task is cleared after success or failure. This avoids starting a separate refresh for every expired authenticated request.

[Architecture](ENGINEERING.md) · [Data model](DATA_MODEL.md)
