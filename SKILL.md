---
name: lit-best-practices
description: "Enforce Lit web component best practices covering reactive property declaration, template rendering, CSS encapsulation with static styles, lifecycle callback ordering, custom event composition, ARIA accessibility, and rendering performance optimization. Use when writing, reviewing, or refactoring Lit components, custom elements, shadow DOM templates, reactive properties, or web component performance."
license: MIT
author: community
version: 1.0.0
---

# Lit Web Components Best Practices

27 rules across 7 categories for building Lit web components — optimized for AI-assisted code generation, review, and refactoring.

## How to Apply These Rules

1. **When reviewing code:** Check all CRITICAL rules first, then HIGH rules relevant to changed code
2. **When writing new components:** Load rules for Component Structure (1-x), Styling (3-1), and Events (4-1)
3. **When optimizing performance:** Load rules for Rendering (2-x) and Performance (7-x)
4. **When auditing accessibility:** Load rules 6-1 through 6-3

## Rules Index

### 1. Component Structure

- `rules/1-1-use-decorators.md` — Use TypeScript Decorators (HIGH): declare properties with `@property()` and `@state()` decorators instead of static `properties` block
- `rules/1-2-separate-state.md` — Separate Public Properties from Internal State (HIGH): use `@property()` for public API, `@state() private` for internal values
- `rules/1-3-reflect-sparingly.md` — Reflect Properties Sparingly (MEDIUM): only set `reflect: true` on primitives used in CSS selectors; never reflect objects/arrays
- `rules/1-4-default-values.md` — Always Provide Default Values (HIGH): every `@property()` and `@state()` must have a default to avoid `undefined` in templates

### 2. Rendering

- `rules/2-1-pure-render.md` — Keep render() Pure (CRITICAL): `render()` must be side-effect free — no property mutations, no dispatched events, no console output
- `rules/2-2-use-nothing.md` — Use `nothing` for Empty Content (MEDIUM): return `nothing` instead of empty string or `null` to avoid rendering empty text nodes
- `rules/2-3-use-repeat.md` — Use repeat() for Keyed Lists (HIGH): use `repeat(items, keyFn, templateFn)` instead of `map()` to preserve DOM identity on reorder
- `rules/2-4-use-cache.md` — Use cache() for Conditional Subtrees (MEDIUM): wrap toggled template blocks in `cache()` to preserve DOM state across show/hide cycles
- `rules/2-5-derived-state.md` — Compute Derived State in willUpdate() (HIGH): calculate derived values in `willUpdate()`, not `render()` or `updated()`

### 3. Styling

- `rules/3-1-static-styles.md` — Always Use Static Styles (CRITICAL): use `static styles = css\`...\`` — never inline `<style>` tags inside templates
- `rules/3-2-host-styling.md` — Style the Host Element Properly (HIGH): always include `:host { display: block; }` and `:host([hidden]) { display: none; }`
- `rules/3-3-css-custom-properties.md` — CSS Custom Properties for Theming (MEDIUM): expose `--component-*` custom properties for external theming
- `rules/3-4-css-parts.md` — CSS Parts for Deep Styling (MEDIUM): add `part="name"` attributes to key inner elements for `::part()` styling

### 4. Events

- `rules/4-1-composed-events.md` — Dispatch Composed Events (CRITICAL): always set `{ bubbles: true, composed: true }` so events cross shadow DOM boundaries
- `rules/4-2-event-naming.md` — Event Naming Conventions (MEDIUM): use lowercase kebab-case names prefixed with component name for custom events
- `rules/4-3-cleanup-listeners.md` — Clean Up Event Listeners (HIGH): remove manual listeners in `disconnectedCallback()` to prevent memory leaks

### 5. Lifecycle

- `rules/5-1-super-call-order.md` — Correct super() Call Order (CRITICAL): call `super.connectedCallback()` first, `super.disconnectedCallback()` last
- `rules/5-2-first-updated.md` — Use firstUpdated for DOM Operations (HIGH): access shadow DOM nodes in `firstUpdated()`, not `connectedCallback()`
- `rules/5-3-will-update.md` — Use willUpdate for Derived State (HIGH): compute derived values in `willUpdate(changedProperties)` before render
- `rules/5-4-update-complete.md` — Async Operations with updateComplete (MEDIUM): await `this.updateComplete` before measuring DOM after property changes

### 6. Accessibility

- `rules/6-1-delegates-focus.md` — delegatesFocus for Interactive Components (HIGH): set `static shadowRootOptions = { ...LitElement.shadowRootOptions, delegatesFocus: true }`
- `rules/6-2-aria-attributes.md` — ARIA for Custom Interactive Components (CRITICAL): add `role`, `tabindex`, `aria-*` attributes and keyboard handlers (Space/Enter) to interactive elements
- `rules/6-3-form-associated.md` — Form-Associated Custom Elements (HIGH): set `static formAssociated = true` and use `ElementInternals` for form participation

### 7. Performance

- `rules/7-1-has-changed.md` — Custom hasChanged for Complex Types (HIGH): provide `hasChanged(newVal, oldVal)` for object/array properties to avoid unnecessary re-renders
- `rules/7-2-batch-updates.md` — Batch Property Updates (MEDIUM): set multiple properties synchronously — Lit batches them into a single update cycle
- `rules/7-3-lazy-loading.md` — Lazy Load Heavy Dependencies (HIGH): use dynamic `import()` in `firstUpdated()` or on user interaction for heavy modules
- `rules/7-4-memoization.md` — Memoize Expensive Computations (MEDIUM): cache expensive derived values and invalidate only when source properties change

## CRITICAL Rules — Inline Examples

### Pure render() (2-1)

```typescript
// ✗ Side effects in render
render() { this.count++; return html`...`; }

// ✓ Pure — only return templates
render() { return html`<span>${this.count}</span>`; }
```

### Static styles only (3-1)

```typescript
// ✗ Never inline <style> in templates
render() { return html`<style>:host{color:red}</style>...`; }

// ✓ Always use static styles
static styles = css`:host { color: red; }`;
```

### Composed events (4-1)

```typescript
// ✗ Event won't cross shadow DOM
this.dispatchEvent(new CustomEvent('change', { detail: val }));

// ✓ Set bubbles + composed
this.dispatchEvent(new CustomEvent('change', {
  bubbles: true, composed: true, detail: val
}));
```

### super() call order (5-1)

```typescript
connectedCallback() {
  super.connectedCallback(); // ← FIRST
  this.addEventListener('click', this._onClick);
}
disconnectedCallback() {
  this.removeEventListener('click', this._onClick);
  super.disconnectedCallback(); // ← LAST
}
```

### ARIA on interactive elements (6-2)

```typescript
// ✓ Add role, tabindex, and keyboard handling
render() {
  return html`<div role="button" tabindex="0"
    @click=${this._handleClick}
    @keydown=${this._handleKeydown}>${this.label}</div>`;
}
```

## Component Skeleton

```typescript
@customElement('my-component')
export class MyComponent extends LitElement {
  static styles = css`
    :host { display: block; }
    :host([hidden]) { display: none; }
  `;

  @property({ type: String }) value = '';
  @property({ type: Boolean, reflect: true }) disabled = false;
  @state() private _internal = '';

  render() {
    return html`<slot></slot>`;
  }
}
```

## Verification Checklist

After writing or refactoring a Lit component, verify:

- [ ] `render()` contains no side effects (no mutations, no events, no logging)
- [ ] Styles use `static styles = css\`...\`` — no `<style>` in templates
- [ ] Custom events set `bubbles: true, composed: true`
- [ ] `super.connectedCallback()` called first; `super.disconnectedCallback()` called last
- [ ] Interactive elements have `role`, `tabindex`, and keyboard handlers
- [ ] All `@property()` and `@state()` have default values
- [ ] `:host { display: block; }` and `:host([hidden]) { display: none; }` defined
