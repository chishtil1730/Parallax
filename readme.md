# Z-Lens

## Cross-Framework Z-Index Intelligence & Visualization

---

## 1. Problem Statement

In modern web applications, `z-index` often becomes a silent source of complexity and bugs:

* Arbitrary values like `z-index: 9999`
* Implicit stacking contexts created by `position`, `transform`, `opacity`, `filter`, etc.
* Framework abstractions (React, Tailwind, CSS-in-JS) hiding actual browser behavior
* Debugging that relies on trial-and-error rather than system understanding

Existing browser DevTools expose *computed styles*, but they lack:

* A global stacking overview
* Source-level traceability
* Safe refactoring workflows
* Change history and intent

---

## Key Features

| Feature | Description |
|-------|-------------|
| Runtime Stacking Inspector | Detects actual stacking contexts and computed z-index values directly from the browser |
| 3D Layer Visualization | Displays stacking contexts and elements along an X-axis to reveal depth and overlap |
| Cross-Framework Support | Works across HTML, CSS, React, Next.js, Tailwind, and CSS-in-JS via resolvers |
| Source Mapping | Maps rendered elements back to their exact source files and definitions |
| Editable Z-Index Table | Centralized table to view and modify all z-index definitions safely |
| Git-Style Staging | Stage, review, and commit z-index changes with diffs |
| Non-Destructive Edits | Changes are applied via patches, not blind file mutation |
| Framework-Agnostic Core | Built on browser stacking behavior, not framework-specific assumptions |


## 3D Pan Visualization

The 3D pan view represents the browser’s stacking model in spatial form.

- The **X-axis** represents stacking depth (z-order)
- Each **stacking context** is rendered as a separate plane
- Elements within a context appear as nodes positioned relative to their parent
- Panning along the X-axis reveals overlap and render order visually

Interactions:
- Hovering an element highlights it in the live page
- Clicking focuses its stacking context and source mapping
- Context boundaries make implicit stacking rules explicit

This visualization explains *why* elements overlap the way they do, not just *that* they overlap.

## Enhanced Features

| Feature | Description | Status |
|---------|-------------|--------|
| **Design Token Recognition** | Detects and works with existing CSS variables, design tokens, and constants | Enhanced v1 |
| **CVA/Variant Integration** | Recognizes and can edit class-variance-authority and similar styling systems | Enhanced v1 |
| **Smart Table Grouping** | Group z-index values by stacking context, file, framework type, or custom groups | Enhanced v1 |
| **Visual Before/After Diff** | Shows not just code changes but visual preview of how the page will look | Enhanced v1 |
| **Named Layer Contracts** | Define and enforce semantic layers (e.g., `modal > overlay > tooltip`) | Future/v2 |

## AI-Powered Features

| Feature | Description | Status |
|---------|-------------|--------|
| **Automatic Issue Detection** | AI identifies common z-index anti-patterns (multiple 9999s, broken stacking contexts) | AI Feature |
| **Intelligent Fix Suggestions** | Context-aware recommendations for resolving stacking issues | AI Feature |
| **Layering System Generator** | AI suggests and generates design token/constant systems based on usage patterns | AI Feature |
| **Code Migration Assistant** | Generates patches to refactor scattered z-indices into organized systems | AI Feature |
| **Interactive AI Chat** | Ask follow-up questions about specific stacking issues with full context | AI Feature |
| **One-Click Apply Suggestions** | Apply AI-recommended fixes through the staging workflow | AI Feature |

## Technical Capabilities

| Capability | Description | Status |
|------------|-------------|--------|
| **Framework-Agnostic Core** | Browser stacking behavior is source of truth, frameworks are input formats | Core v1 |
| **Pluggable Resolver System** | Extensible architecture for adding new framework/styling system support | Core v1 |
| **Privacy-First AI Integration** | Only sends structural CSS data to AI, not file contents or user data | AI Feature |
| **Graceful Failure** | Tool remains functional even when source mapping fails | Core v1 |
| **Browser Extension Architecture** | Works without requiring code integration or build step changes | Core v1 |

## Future Extensions

| Feature | Description | Status |
|---------|-------------|--------|
| **Z-Index Budgets** | Set and enforce maximum z-index values per layer/context | Future |
| **Team Conventions** | Define and share z-index conventions across team members | Future |
| **CI/CD Integration** | Automated warnings for stacking violations in pull requests | Future |
| **Shadow DOM Support** | Handle Web Components and Shadow DOM stacking contexts | Future |
| **Dynamic Z-Index Tracking** | Track z-indices set via JS state, animations, and scroll positions | Future |
| **Media Query Awareness** | Handle different z-index values across breakpoints | Future |

## 2. Core Idea

**Z-Lens treats z-index as a system, not a single CSS property.**

It combines:

1. Runtime inspection of stacking contexts (browser truth)
2. Static analysis of the codebase (source truth)
3. Git-style staging and committing of z-index changes

The goal is to make layering behavior **visible, explainable, and controllable** across frameworks.

---

## 3. Design Principles

* **Browser behavior is the source of truth**
* Frameworks are treated as *input formats*, not core logic
* Runtime data and source data must be linkable but independent
* Changes should be intentional, reviewable, and reversible

---

## 4. High-Level Architecture

Z-Lens is composed of four cooperating layers:

1. **Runtime Inspector (Browser)**
   Observes computed CSS and DOM stacking behavior

2. **Visualization Layer**
   Presents stacking contexts and elements in a 3D spatial model

3. **Source Mapping Engine**
   Maps runtime elements back to framework-specific source locations

4. **Patch & Staging Engine**
   Applies controlled changes to the codebase with diffs and commits

---

## 5. Runtime Layer (Language-Agnostic)

This layer runs inside the browser (extension / devtools panel) and does **not** depend on how the code was written.

### Responsibilities

* Walk the DOM tree
* Read computed styles using `getComputedStyle`
* Detect stacking context creators:

  * `position` + non-auto `z-index`
  * `transform`
  * `opacity < 1`
  * `filter`
  * `isolation: isolate`
* Build a **stacking context graph**

This graph represents *actual rendering order*, not declared order.

---

## 6. Visualization Layer (3D Layer View)

The visualization presents stacking contexts in a spatial model:

* X-axis represents depth / stacking order
* Each stacking context is a plane
* Elements are rendered as nodes/cards within planes
* Hover highlights the element in the page
* Click focuses the source mapping

This answers questions like:

* Why is this element above another?
* Which stacking context controls this element?
* Why does changing z-index have no effect?

---

## 7. Source Mapping Engine (Framework-Aware)

Z-Lens uses **pluggable resolvers** to map runtime elements back to source code.

### Resolver Responsibilities

* Identify how a z-index was defined
* Locate its source file and position
* Provide a safe edit strategy

### Supported Sources (Extensible)

* Plain CSS / HTML
* React / JSX / TSX
* Tailwind CSS utilities and config
* CSS-in-JS (styled-components, emotion)
* Inline styles

If mapping fails, runtime data remains usable.

---

## 8. Editing Model

Z-Lens does not directly mutate files.

Instead, it:

* Generates patches
* Shows diffs (before/after)
* Allows selective staging of changes
* Applies changes only on user confirmation

Each table row knows *how* it can be safely edited based on its resolver.

---

## 9. Git-Style Workflow

Changes flow through stages:

1. Inspect runtime stacking
2. Propose z-index adjustments
3. Stage selected changes
4. Review diffs
5. Commit or discard

This prevents accidental layout regressions and encourages intentional layering design.

---

## 10. Why This Scales

* Framework-agnostic core
* Plugin-based source resolvers
* Browser-defined behavior guarantees correctness
* Encourages design-system-level thinking for layers

Z-Lens turns a historically ad-hoc CSS problem into a **structured, debuggable system**.

---

## 11. Future Extensions

* Named layer contracts (e.g. `modal > overlay > tooltip`)
* Z-index budgets and constraints
* Team-wide layering conventions
* CI warnings for stacking violations
* Shadow DOM and Web Components support
