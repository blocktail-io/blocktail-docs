# Behavioral Patterns

Blocktail introduces two behavioral patterns—`observer` and `agent`—on top of the foundational L1–L5 layers (`matrix`, `context`, `block`, `mutation`, `tail`) that we previously discussed. While those five layers define where components live, Observers and Agents define how they behave. This system provides a structured way to handle block-level lifecycle (Observers) and cross-layer behaviors (Agents) without violating Blocktail's semantic boundaries.

## Understanding the Behavioral Layer System

### Observer (B1) and Agent (B2)

- **B1: Observer** (*Behavioral Sovereignty Layer*)
  - **Marker:** `data-observer="{ObserverName}"`
  - **Purpose:** Owns the entire block lifecycle, states, and resources; ensures block-level sovereignty

- **B2: Agent** (*Behavioral Application Layer*)
  - **Marker:** `data-agent="{AgentName}"`
  - **Purpose:** Attaches modular or cross-layer behaviors (e.g., analytics, animations) without claiming block sovereignty

By separating block-level ownership (Observer) from modular utilities (Agent), we preserve clear boundaries. Observers control all block logic, yet Agents remain reusable across any layer.

## B1: Behavioral Sovereignty Layer (Observer)

### Marker

```
data-observer="{ObserverName}"
```

### Purpose

An Observer manages all lifecycle events, state transitions (mutations), resource usage, and child element interactions for a single block.

### Boundary

- Operates only at the block level (L3)
- Claims full control over the block's mutations and resource governance
- One Observer per block (no shared sovereignty)
- Establishes a root for processing the block's behavior

### Scope

- Governs state transitions, e.g., toggling `--featured`, `--on_sale`
- Manages child elements and event handlers within the block boundary
- Maintains resource lifecycle (listeners, network calls)

#### Composition Model

1. One observer per block—never mount multiple observers on the same block or violate the sister block's sovereignty.
2. Observers may invoke Agents or utility modules (e.g., analytics, logging) internally when extra functionality is needed
3. **Observer-First:** All lifecycle logic stays in the main observer. Agents or helper modules remain add-ons but do not become second observers
4. **Clear Sovereignty:** Avoid sub-observers in the DOM—any specialized or repeated tasks can be delegated to an Agent or a utility imported directly into the observer

> **Behavioral Sovereignty terminology:** The notion that one entity (the Observer) is the single authority over a block's states, resources, and lifecycle.

## B2: Behavioral Application Layer (Agent)

### Marker

```
data-agent="{AgentName}"
```

### Purpose

Applies standardized or reusable behaviors across any layer (L1–L5). Agents may be:
- Mounted via `data-agent` (e.g., `data-agent="LazyLoad"`)
- Invoked by an Observer for cross-layer tasks (analytics, animations, global updates, logging)

### Boundary

- No sovereignty claims over the block
- Practically enhances or augments any layer: system, context, block, or tail
- Coexists with Observers but never competes for state or lifecycle

### Scope

- Integrates third-party functionality or custom utility scripts
- Maintains pattern consistency by separating cross-layer behaviors from block-specific logic

> **Behavioral Application terminology:** A utility or cross-layer behavior that applies enhancements without claiming ownership.

## Layer Integration Matrix

Let's take a look at how B1 (Observer) and B2 (Agent) integrate with the core L1–L5 layers:

### System Context Layer (L1: matrix--)

```html
<!-- L1 + B2: System-wide Agent -->
<body class="matrix--shop"                 <!-- L1 -->
      data-agent="SystemAnalytics">        <!-- B2: global analytics -->
  <!-- System-wide behavior application -->
</body>

/* ✗ INVALID: Observers at system level */
<body class="matrix--shop"
      data-observer="SystemObserver">      <!-- Invalid -->
</body>
```

### Local Context Layer (L2: context--)

```html
<!-- L2 + B2: Context Agent -->
<main class="context--catalog"             <!-- L2 -->
      data-agent="InfiniteScroll">         <!-- B2 -->
  <!-- Context-level behavior application -->
</main>

/* ✗ INVALID: Observers at context level */
<main class="context--catalog"
      data-observer="CatalogObserver">     <!-- Invalid -->
</main>
```

### Component Reference Layer (L3: block_name)

```html
<!-- L3 + B1: Block Observer with Mutations -->
<article class="product_card --featured"   <!-- L3 + L4 -->
         data-observer="ProductObserver">  <!-- B1 -->
  <!-- Observer controls block lifecycle & states -->
</article>

<!-- L3 + B2: Block Agent -->
<article class="product_card"              <!-- L3 -->
         data-agent="RevealAnimation">     <!-- B2 -->
  <!-- Reusable cross-layer behavior at the block level -->
</article>

<!-- L3 + B1 + B2: Combined with Mutations -->
<article class="product_card --featured    <!-- L3 + L4 -->
         --on_sale"                       <!-- Additional mutation -->
         data-observer="ProductObserver"   <!-- B1 -->
         data-agent="RevealAnimation">     <!-- B2 -->
  <!-- Observer retains sovereignty while Agent adds behavior -->
</article>
```

### Element Layer (L5: -tail)

```html
<!-- L5 + B2: Tail Agent -->
<article class="product_card"              <!-- L3 -->
         data-observer="ProductObserver">   <!-- B1 -->
  <div class="-content"                    <!-- L5 -->
       data-agent="LazyLoad">             <!-- B2 -->
    <!-- Tail-level behavior application -->
  </div>
</article>

/* ✗ INVALID: Observers at tail level */
<article class="product_card">             <!-- L3 -->
  <div class="-content"                    <!-- L5 -->
       data-observer="ContentObserver">    <!-- Invalid -->
  </div>
</article>
```

## Layer Access Rules

### Observer

```html
<!-- ✓ VALID: Block-Level with Mutations -->
<article class="product_card --featured --on_sale" 
         data-observer="ProductObserver">
  <!-- Observer controls the entire block (states, resources) -->
  <div class="-content">
    <!-- Everything is under observer control -->
  </div>
</article>

/* ✗ INVALID: Observers at other layers */
<body class="matrix--shop" data-observer="ShopObserver">         <!-- Invalid -->
<main class="context--catalog" data-observer="CatalogObserver">  <!-- Invalid -->
<div class="-content" data-observer="ContentObserver">          <!-- Invalid -->
```

### Agent

```html
<!-- ✓ VALID: Agents can attach to any layer -->
<body class="matrix--shop" data-agent="SystemAgent">          <!-- L1 -->
<main class="context--catalog" data-agent="ContextAgent">     <!-- L2 -->
<article class="product_card" data-agent="BlockAgent">        <!-- L3 -->
<div class="-content" data-agent="TailAgent">                 <!-- L5 -->
```

## Observer Mutation Control

Observers manage all block-level mutations (e.g. `--active`, `--featured`) without interference:

```javascript
class ProductObserver {
  constructor(element) {
    this.block = element;
    this.mutations = this.getCurrentMutations();
  }

  getCurrentMutations() {
    return {
      featured: this.block.classList.contains('--featured'),
      onSale:   this.block.classList.contains('--on_sale')
    };
  }

  init() {
    if (this.mutations.featured) this.initializeFeaturedState();
    if (this.mutations.onSale)   this.initializeSaleState();
  }

  setMutation(mutation, active) {
    const mutClass = `--${mutation}`;
    this.block.classList.toggle(mutClass, active);
    this.mutations[mutation] = active;
    active ? this.initializeMutationState(mutation)
           : this.cleanupMutationState(mutation);
  }
}
```

This approach:
1. **Enforces** single ownership of states
2. **Keeps** block-level transitions consistent
3. **Preserves** pattern recognition across expansions

## Pattern Recognition Flow

```html
<PATTERN_FLOW>
matrix--shop                     <!-- L1: system -->
  data-agent="Analytics"         <!-- Agent for system-wide analytics -->
  context--catalog               <!-- L2: context -->
    data-agent="InfiniteScroll"  <!-- Agent for context-level pagination -->
    product_card                 <!-- L3: block -->
      --featured --on_sale       <!-- L4: mutations -->
      data-observer="ProductObserver"   <!-- Observer = block sovereignty -->
      data-agent="RevealAnimation"      <!-- Agent = cross-layer effect -->
      -content                   <!-- L5: tail -->
        data-agent="LazyLoad"    <!-- Additional tail-level agent -->
</PATTERN_FLOW>
```

## Implementation Benefits

1. **Clear Behavioral Boundaries**
   - Observer for block sovereignty, Agent for modular behaviors

2. **State Management**
   - Observers unify block-level state transitions
   - Agents add or remove cross-layer tasks easily

3. **Pattern Recognition Stability**
   - Minimal collisions, fewer naming collisions or confusion

4. **Predictable Scaling**
   - Observers handle complex block expansions; Agents remain reusable

5. **Maintainable Structure**
   - Single responsibility for block logic, flexible agent invocation

## Key Takeaways

1. **Observer (B1):** Block sovereignty: all lifecycle, states, resource usage in a single owner
2. **Agent (B2):** Cross-layer or reusable utilities: attachable at any layer, never overshadowing the observer

This Behavioral Layer System ensures semantic clarity, predictable evolution, and scalable mission-critical architecture. Observers keep each block's logic unified, while Agents offer flexible, reusable enhancements with minimal overhead or fragmentation—a crucial balance for large or fast-evolving front-end codebases.



# Data Observer

A Data Observer provides block-level behavioral sovereignty through its `data-observer` attribute, ensuring one observer per block and granting that observer full control over the block's lifecycle, states, and resource management.

> **Core Principle:** A block's Observer is its single controller, unifying lifecycle events, mutation states, and resource usage in one place.

## Introduction

A Data Observer is a behavioral pattern that initializes like a module to govern a block's interactions, states, and lifecycle. This approach bridges the gap between static HTML and dynamic interactions by providing a centralized solution for managing all complex behaviors in web applications.

- **Single Observer Per Block:** No second observer can attach to the same block. This prevents collisions or confusion.
- **Observer-First:** All block-level logic (mutations, resource usage, event handling) belongs in the observer.
- **Optional Composition:** When you need extra specialized behaviors (e.g., analytics or logs tracking), you embed them inside the main observer, maintaining a one-observer rule in the DOM.

## Why Use an Observer?

1. Consolidates logic for state transitions (`--mutation`) and resource usage into one place.
2. By disallowing multiple observers on the same block, overlapping lifecycle management is avoided.
3. Observers act as a single "source of truth," simplifying debugging, iteration, and generative AI-driven expansions.

> **Observation Point:** `data-observer` is the block-level marker that attaches an "owner" class to manage the entire block. This design separates structure from behavior while preserving clear relationships between components and dynamic functionality.

## Marker Reference

```html
<!-- Basic usage -->
<article class="product_card"
         data-observer="MyCustomObserver">
  <!-- MyCustomObserver JS class now governs this block's lifecycle -->
</article>
```

- **Syntax:** `data-observer="ObserverName"`
- **ObserverName:** Typically, the ES6 class or function name representing block logic (e.g., `MediaPlayerObserver`, `ProductObserver`).

## Purpose & Boundaries

### Purpose

- **Lifecycle Control:** Initialize, update, and destroy block resources.
- **Mutation Management:** Toggling or handling block states (`--featured`, `--active`), ensuring consistent transitions.
- **Resource Governance:** Manage network calls, event listeners, or local data relevant to this block.

### Boundaries

- **Scope:** Observers operate only at the block level (L3 in Blocktail).
- **Sovereignty:** Only controls the block it's mounted on.
- **Layer Rules:**
  - Invalid at L1 (Matrix)
  - Invalid at L2 (Context)
  - Invalid at L5 (Tail)

An example of an invalid attempt would be:

```html
<!-- WRONG: Observers can't attach to a Context -->
<main class="context--catalog"
      data-observer="CatalogObserver">  <!-- Invalid -->
</main>
```

## Single Observer Principle

### Traditional JavaScript Module Approach

Outside of Blocktail, developers often rely on "`js-*`" selectors or ad-hoc modules for dynamic behaviors:

```html
<!-- Traditional Pattern -->
<div class="js-slider js-animated js-lazy">
  <div class="js-slider-content">
    <div class="js-slider-item">
      <!-- Pattern confusion & multiplication -->
    </div>
  </div>
</div>
```

#### A few key issues with the approach:

1. **Pattern Degradation:** Classes and scripts accumulate, losing clarity over time
2. **Unclear Behavioral Ownership:** Different modules may override each other's logic
3. **Scattered State Management:** States (e.g., active, lazy) spread across multiple classes
4. **Unreliable Resource Cleanup:** Hard to track who sets or removes event listeners

### Why Single Observer?

In Blocktail, one `data-observer` per block ensures a single source of truth for all relevant lifecycle events, state management (mutations), resource usage, and child elements. If additional specialized behaviors—like analytics or performance tracking—are required, you can:

- Embed them as internal modules within the observer's code, or
- Invoke Agents (`data-agent`), either mounted in the HTML or dynamically called from the observer for cross-layer tasks

> **Key Point:** Centralizing all block logic in a single observer prevents collisions and confusion. The HTML should reflect one block owner, ensuring immediate clarity about "who controls this block."

### Composition Example

```javascript
class AnalyticsModule {
  constructor(parentObserver) {
    this.parentObserver = parentObserver;
  }
  init() { /* Analytics setup */ }
  trackEvent(eventName) { /* Logging logic */ }
  cleanup() { /* Remove listeners */ }
}

class PerformanceModule {
  constructor(parentObserver) {
    this.parentObserver = parentObserver;
  }
  init() { /* Performance metrics init */ }
  monitor() { /* Track memory usage, CPU stats, etc. */ }
  cleanup() { /* Dispose of resources */ }
}

class MediaPlayerObserver {
  constructor(element) {
    this.block = element;
    // Internal modules for specialized tasks
    this.analytics = new AnalyticsModule(this);
    this.performance = new PerformanceModule(this);
  }

  init() {
    // Main block setup
    this.initializePlayer();
    
    // Initialize internal modules
    this.analytics.init();
    this.performance.init();
  }

  onPlay() {
    this.player.play();
    // Delegate specialized logic to internal modules
    this.analytics.trackEvent('play');
    this.performance.monitor();
    
    // Alternatively, if cross-layer or repeated logic is needed:
    // AgentRegistry.invoke('ParallaxScroll', { element: this.block });
  }

  destroy() {
    // Cleanup internal modules first
    this.analytics.cleanup();
    this.performance.cleanup();
    // Then finalize main observer tasks
  }
}
```

- `MediaPlayerObserver` is the single observer attached in the DOM:

```html
<article class="media_player"
         data-observer="MediaPlayerObserver">
```

- **AnalyticsModule** and **PerformanceModule** are internal modules. They do not appear as second observers.
- Agents can also be invoked from code if the block needs cross-layer or repeated behaviors (e.g., parallax effect).

### Advantages

1. **Single Observer in the DOM**
   - No second `data-observer="Xyz"` on the same block—only one observer claims sovereignty.

2. **No Sovereignty Confusion**
   - Internal modules or agent invocations never create additional block owners.

3. **Reusability**
   - If you need the same analytics logic in another block, simply reuse the same internal module or agent invocation in that block's observer.

4. **Reduced HTML Noise**
   - The markup has only one observer attribute, protecting us from "JS keyword searches" for finding all controlling scripts.

## State (Mutation) Management

Observers typically handle all block-specific states (mutations). For instance:

```javascript
class ProductObserver {
  constructor(element) {
    this.block = element;
    this.mutations = this.getCurrentMutations();
  }

  getCurrentMutations() {
    return {
      isFeatured: this.block.classList.contains('--featured'),
      isOnSale:   this.block.classList.contains('--on_sale')
    };
  }

  init() {
    if (this.mutations.isFeatured) {
      this.activateFeaturedState();
    }
    if (this.mutations.isOnSale) {
      this.activateOnSaleState();
    }
  }

  setMutation(mutation, active) {
    const className = `--${mutation}`;
    this.block.classList.toggle(className, active);
    this.mutations[`is${mutation[0].toUpperCase() + mutation.slice(1)}`] = active;

    if (active) {
      this[`activate${mutation[0].toUpperCase() + mutation.slice(1)}State`]();
    } else {
      this[`deactivate${mutation[0].toUpperCase() + mutation.slice(1)}State`]();
    }
  }
}
```

1. Localize block-specific states within the observer.
2. Methods like `activateFeaturedState()` maintain clarity, avoiding random toggles throughout the code.

### Example Usage in HTML

```html
<article class="product_card --featured --on_sale"
         data-observer="ProductObserver">
  <h2 class="-title">Sample Product</h2>
  <div class="-price_container">
    <span class="-original">$99</span>
    <span class="-sale">$79</span>
  </div>
</article>
```

- The `product_card` block has a single `data-observer="ProductObserver"`.
- States `--featured`, `--on_sale` are toggled by that observer's code.

## Runtime Adaptation

When mounted, an `observer` acknowledges pre-existing mutations from server-side rendering or static compilation and then maintains control over subsequent state changes.

### Initial State Recognition

```javascript
class MediaPlayerObserver {
  constructor(element) {
    this.block = element;
    // Capture pre-loaded states
    this.mutations = {
      isLightMode: this.block.classList.contains('--light_mode'),
      isAutoplay: this.block.classList.contains('--autoplay'), 
      isPreferred: this.block.classList.contains('--preferred'),
      isVideo: this.block.classList.contains('--video'),
      isAudio: this.block.classList.contains('--audio')
      /* ... */
    };
  }

  init() {
    // Initialize based on existing mutations
    if (this.mutations.isVideo) {
      this.initializeVideoPlayer();
    } else if (this.mutations.isAudio) {
      this.initializeAudioPlayer(); 
    }
    if (this.mutations.isAutoplay) {
      this.setupAutoplay();
    }
  }
}
```

- **Initial State:** Observers respect mutations present before initialization, such as server-rendered personalization.
- **State Management:** After initialization, mutation changes are handled through observer methods to ensure consistency.

## Common DOs and DON'Ts

**DO:**
- Compose additional logic internally for advanced features.
- Manage block states (mutations) in the observer.
- Centralize block lifecycle (init, transitions, destroy) in the observer while leveraging Agents for cross-layer utility tasks.

**DON'T:**
- Attach an observer to L1 (matrix), L2 (context), or L5 (tail).
- Mount multiple observers on the same block.
- Manage another block's states from your observer (no cross-block meddling, please).
- Scatter resource usage or events in multiple watchers—centralize them in the single observer.

## Relationship to Agents (data-agent)

- Agents can appear at any layer (L1–L5) for cross-layer or reusable tasks (e.g., lazy loading, infinite scroll).
- Unlike an Observer, an Agent never manages the block's lifecycle or states.
- An Observer and an Agent can coexist on the same block:

```html
<article class="product_card --featured"
         data-observer="ProductObserver"
         data-agent="RevealAnimation">
  <!-- Observer = lifecycle owner, Agent = animation effect -->
</article>
```

- Observers remain sovereign over the block, while Agents handle small "addon" like behaviors or repeated utility tasks.

## Key Takeaways

- Additional sub-observers (analytics, performance, etc.) can be embedded to handle specialized logic—all within the main observer.
- Observers operate at L3 only, ensuring alignment with Blocktail's layering principles.
- Observers unify our block logic, simplifying debugging and iterative expansions.
- Cross-layer or repeated behaviors are attached as Agents, but never overshadow the observer's control.

For a top-level view of how Observers (B1) and Agents (B2) fit into the entire Behavioral Layer System—including dynamic invocation vs. mounting—see the [Behavioral Layer System](https://blocktail.dev/fundamentals/behavioral-patterns) page. Next, continue reading about [Data Agent](https://blocktail.dev/methodology/data-agent), which handles cross-layer or repeated behaviors without interfering in block-level sovereignty.


# Data Agent

A Data Agent offers utility and cross-layer behaviors through its `data-agent` attribute, providing additive enhancements at any layer (L1–L5) without claiming block-level ownership or lifecycle control. Agents can handle tasks like animations, lazy loading, or analytics, leaving state management and the block sovereignty to an Observer, if one is present.

## Why Use an Agent?

1. **Cross-Layer Flexibility**
   - Attach a behavior at any layer—system (L1), local context (L2), block (L3), or tail (L5).
   - Example: `data-agent="Analytics"` on `<body>` for global tracking, or `data-agent="LazyLoad"` on a tail element.

2. **Plug-and-Play**
   - Agents typically contain a self-contained feature—like a fade-in effect or analytics snippet—that can be quickly applied or removed.

3. **No Lifecycle Collisions**
   - Agents do not manage block lifecycle or states, so they never conflict with Observers.

## Marker Reference

```html
<!-- Basic usage -->
<div class="-content"
     data-agent="LazyLoad">
  <!-- The LazyLoad agent handles lazy loading of images or content -->
</div>
```

- **Syntax:** `data-agent="AgentName"`
- **AgentName:** A descriptive identifier for the behavior (e.g., `RevealAnimation`, `InfiniteScroll`, `LazyLoad`).

## Purpose & Boundaries

### Purpose

- **Standardized Behaviors:** Provide a uniform approach for common tasks like animations, logging, or data loading.
- **Utility Layer:** Agents can be dropped in to handle cross-layer or traditional repeated behaviors without rewriting the block's main logic.
- **Framework Independence:** Because Agents lack block ownership, they easily integrate external libraries or utility scripts.

### Boundaries

1. **No Sovereignty**
   - Agents never manage or override a block's lifecycle, states, or resource usage—that's the Observer's job.

2. **Valid on All Layers**
   - **System** (L1, `matrix--shop`)
   - **Local Context** (L2, `context--catalog`)
   - **Block** (L3, `product_card`)
   - **Mutation** (L4, `--featured`) — Agents may respond visually, but do not define states
   - **Tail** (L5, `-content`)

3. **Coexists with Observers**
   - Agents never override an Observer's ownership of the block.
   - *Example (invalid):* An agent tries to "take over" block states:
   ```html
   <!-- Wrong: Agents trying to mutate block ownership or states -->
   <div class="product_card"
        data-agent="BlockOwnershipChanger">
     <!-- There's no concept of 'BlockOwnershipChanger' -->
   </div>
   ```
   - Agents should enhance or add behaviors, not shift state control from the Observer.

## Agent Scope & Examples

### Cross-Layer Scope

Agents can appear:

1. **System Layer (L1)**
   - `<body class="matrix--shop" data-agent="Analytics">` for global analytics

2. **Context Layer (L2)**
   - `<main class="context--catalog" data-agent="InfiniteScroll">` for context-level infinite scrolling

3. **Block Layer (L3)**
   - `<article class="product_card" data-agent="RevealAnimation">` for repeated animations

4. **Tail Layer (L5)**
   - `<div class="-image" data-agent="LazyLoad">` for localized behavior

**Since Agents don't claim ownership,** the same agent can appear multiple times if needed, e.g., multiple `<div data-agent="LazyLoad">` throughout the page.

### Typical Agent Examples

1. **RevealAnimation**
   - Fades in or slides in a block/tail element when it enters the viewport.

2. **InfiniteScroll**
   - Automatically loads more items at the bottom of a listing (often attached at a context layer).

3. **LazyLoad**
   - Defers loading images until they're visible in the viewport.

4. **SystemAnalytics**
   - Attaches a site-wide analytics script to `<body>` for capturing events.

## Example Usage in HTML

```html
<body class="matrix--shop"
      data-agent="SystemAnalytics">
  
  <main class="context--catalog"
        data-agent="InfiniteScroll">
    
    <article class="product_card --featured"
             data-observer="ProductObserver"  <!-- Observer owns block states -->
             data-agent="RevealAnimation">     <!-- Agent adds a reveal effect -->
      <div class="-image"
           data-agent="LazyLoad">             <!-- Lazy loading for images -->
        <img src="product.jpg" />
      </div>
    </article>

  </main>
</body>
```

#### Key Observations:

- `<body>` uses `data-agent="SystemAnalytics"` for global analytics.
- `<main>` uses `data-agent="InfiniteScroll"` for context-level pagination logic.
- `<article>` simultaneously has `data-observer="ProductObserver"` (lifecycle & states) and `data-agent="RevealAnimation"` (visual enhancement).
- A tail element (`-image`) has `data-agent="LazyLoad"`, meaning it only loads images upon scroll.

## DOs and DON'Ts

**DO:**
- Use Agents wherever you need a self-contained feature or utility.
- Attach an agent to any layer without fear of collisions with Observers or other Agents.
- Keep each agent's logic modular (usually a small ES6 class or object).
- Maintain naming consistency—each agent should clearly reflect its purpose (e.g., `LazyLoad`, `RevealAnimation`).

**DON'T:**
- Attempt to manage or override block states or lifecycle from an agent (that belongs to the Observer).
- Attach multiple Agents doing the same job on the same element (risk of unexpected conflicts).

## Relationship with Observers (data-observer)

**Observers** claim block-level lifecycle control, manage states, and ensure consistent resource usage.

**Agents** provide *additional behaviors* at any layer without interfering with the Observer's sovereignty or states.

**Coexistence:**

```html
<article class="product_card --featured"
         data-observer="MediaPlayerObserver"
         data-agent="RevealAnimation">
  <!-- Observer = block-level authority; Agent = cross-layer effect -->
</article>
```

**Result:** Each area of concern (block lifecycle vs. cross-layer behavior) remains separate and conflict-free. Observers focus on block states, and Agents handle repeated or global tasks.

## Key Takeaways

1. Agents can be placed at L1–L5 for reusable behaviors without claiming the block.
2. Because Agents are easily dropped in or invoked, you can add or remove features without rewriting block logic.
3. Teams quickly see an agent's purpose from `data-agent="Xxx"` across the codebase.

`data-agent` attaches utility or cross-layer behaviors at any layer, never usurping the block observer's role. This ensures a clean, predictable layering of advanced front-end features under the Blocktail framework.
