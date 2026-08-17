---
name: component-design
description: Design reusable, composable, well-typed UI components with clear prop contracts, sensible defaults, controlled/uncontrolled patterns, accessibility built in, and minimal coupling; use when creating a new shared component, refactoring a bloated or prop-heavy component, building a design-system primitive, or when users mention component API, props, composition, variants, or reusability.
---

# Component Design

This skill guides the design of UI components that are reusable across contexts, easy to compose, and pleasant to consume. It is framework-agnostic in principle (React, Vue, Svelte, Web Components) while using common patterns that apply broadly.

## When to use this skill

- Creating a new component intended to be reused in more than one place.
- A component has grown too many props, booleans, or conditional branches.
- Building or extending a design system / component library.
- Users mention prop API, composition, variants, slots, controlled vs uncontrolled, or reusability.

## Instructions

1. **Define the single responsibility.** State in one sentence what the component does. If it does several unrelated things, split it. Separate presentational components from ones that own data/logic.
2. **Design the public API before the implementation.** Sketch the props/attributes a consumer will pass. Favor a small, orthogonal set; prefer a `variant`/`size` enum over many booleans that can conflict.
3. **Choose composition over configuration.** Expose slots/children (e.g., `<Card><Card.Header/></Card>`) instead of dozens of props that render sub-parts. Compound components keep the API flat and flexible.
4. **Provide sensible defaults.** The component should look and behave correctly with minimal props. Make the common case one line and the advanced case possible.
5. **Decide controlled vs uncontrolled.** For inputs, support an uncontrolled default (internal state) and an optional controlled mode (`value` + `onChange`). Document which props make it controlled.
6. **Type the contract.** Use TypeScript/PropTypes/schema to make invalid states unrepresentable — discriminated unions for variants, required props where needed, and `readonly` where mutation is not intended.
7. **Forward refs and pass through DOM props.** Spread remaining attributes (`...rest`) onto the root element and forward refs so consumers can style, measure, and attach handlers.
8. **Bake in accessibility.** Use correct roles, labels, focus handling, and keyboard support so consumers get a11y for free (see accessibility-audit).
9. **Keep styling flexible but bounded.** Expose a `className`/`style` escape hatch and design tokens/CSS variables for theming; avoid hard-coded colors and magic numbers.
10. **Document and test.** Provide usage examples, prop tables, and stories. Add tests for the key states, variants, and interactions.

## Examples

Prefer a variant enum over conflicting booleans:

```tsx
// Avoid: booleans can contradict each other (primary + danger?)
<Button primary danger large />

// Prefer: one source of truth per axis
<Button variant="danger" size="lg" />

type ButtonProps = {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
} & React.ButtonHTMLAttributes<HTMLButtonElement>;
```

Compound component using composition instead of many props:

```tsx
<Card>
  <Card.Header>Invoice #1024</Card.Header>
  <Card.Body>Amount due: $240.00</Card.Body>
  <Card.Footer>
    <Button variant="primary">Pay now</Button>
  </Card.Footer>
</Card>
```

Ref forwarding and prop pass-through:

```tsx
const Input = React.forwardRef<HTMLInputElement, InputProps>(
  ({ className, ...rest }, ref) => (
    <input ref={ref} className={cx('input', className)} {...rest} />
  ),
);
```

## Checklist

- [ ] Component has one clear responsibility.
- [ ] Public API designed first; props are minimal and orthogonal.
- [ ] Composition (slots/children) used instead of prop explosion.
- [ ] Sensible defaults; common case needs little config.
- [ ] Controlled and uncontrolled usage decided and documented.
- [ ] Types make invalid states hard to express.
- [ ] Ref forwarded and extra DOM props passed through.
- [ ] Accessibility and keyboard support built in.
- [ ] Styling themeable via tokens with a className escape hatch.
- [ ] Examples, docs, and tests cover key states and variants.
