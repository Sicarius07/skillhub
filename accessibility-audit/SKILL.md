---
name: accessibility-audit
description: Audit and remediate web accessibility issues against WCAG 2.1/2.2 AA — covering semantic HTML, keyboard navigation, focus management, color contrast, ARIA roles, form labels, and screen-reader support; use when reviewing a page or component for a11y, fixing failing audits, preparing for compliance, or when users mention accessibility, WCAG, screen readers, keyboard traps, or contrast.
---

# Accessibility Audit

This skill provides a repeatable process for finding and fixing accessibility defects in web interfaces. It targets WCAG 2.1/2.2 Level AA as the baseline, prioritizing real user impact (keyboard and screen-reader users) over automated-tool score alone.

## When to use this skill

- A page, view, or component needs an accessibility review before shipping.
- An automated scan (axe, Lighthouse, pa11y) reports violations you need to triage and fix.
- Users report they cannot navigate with a keyboard or screen reader.
- You are preparing for a compliance requirement (WCAG, Section 508, EN 301 549, ADA).
- Someone mentions contrast, focus rings, alt text, ARIA, or tab order.

## Instructions

1. **Establish scope and target.** Identify the pages/components under review and confirm the standard (default: WCAG 2.2 AA). List the primary user flows (e.g., sign-up, checkout) — these get the deepest review.
2. **Run automated scans first.** Use axe-core, Lighthouse, or pa11y to catch the ~30-40% of issues that are machine-detectable. Record violations but treat them as a starting point, not the finish line.
3. **Check semantic structure.** Verify one `<h1>` per page and a logical, non-skipping heading order. Confirm landmarks (`<header>`, `<nav>`, `<main>`, `<footer>`) exist and native elements (`<button>`, `<a>`, `<label>`) are used instead of `<div>`/`<span>` with click handlers.
4. **Test keyboard-only.** Unplug the mouse. Tab through every interactive element: confirm all are reachable, focus is visible, order is logical, and there are no keyboard traps. Ensure Enter/Space activate controls and Escape closes overlays.
5. **Verify focus management.** When dialogs/menus open, focus moves into them and is trapped while open; when they close, focus returns to the trigger. Route changes in SPAs should move focus to the new content or heading.
6. **Audit forms.** Every input has an associated `<label>` (or `aria-label`). Errors are announced (`aria-live`, `aria-describedby`), required fields are marked programmatically, and grouping uses `<fieldset>`/`<legend>`.
7. **Check color and contrast.** Text meets 4.5:1 (normal) / 3:1 (large); UI components and focus indicators meet 3:1. Confirm information is never conveyed by color alone.
8. **Review images and media.** Meaningful images have descriptive `alt`; decorative images use `alt=""`. Video has captions; audio has transcripts.
9. **Validate ARIA usage.** Prefer native HTML. Where ARIA is used, confirm roles/states/properties are valid and updated (e.g., `aria-expanded`, `aria-selected`). Remove redundant or conflicting ARIA.
10. **Test with a screen reader.** Use VoiceOver (macOS), NVDA (Windows), or TalkBack (Android) to walk the primary flow and confirm it is understandable.
11. **Prioritize and fix.** Rank findings by severity (blocker → serious → moderate → minor) and user impact, then remediate. Re-test after each fix.

## Examples

Non-semantic clickable div (fails keyboard + screen reader):

```html
<!-- Before: not focusable, no role, no keyboard support -->
<div class="btn" onclick="save()">Save</div>

<!-- After: native button gets focus, keyboard, and role for free -->
<button type="button" onclick="save()">Save</button>
```

Accessible form field with error announcement:

```html
<label for="email">Email address</label>
<input
  id="email"
  name="email"
  type="email"
  aria-describedby="email-error"
  aria-invalid="true"
  required
/>
<p id="email-error" role="alert">Please enter a valid email address.</p>
```

Icon-only button needs an accessible name:

```html
<button type="button" aria-label="Close dialog">
  <svg aria-hidden="true" focusable="false"><!-- x icon --></svg>
</button>
```

## Checklist

- [ ] Automated scan run; violations triaged by severity.
- [ ] Headings form a logical, non-skipping outline; landmarks present.
- [ ] Every flow completable with keyboard only; focus always visible.
- [ ] Focus is trapped in modals and restored on close.
- [ ] All form controls labeled; errors announced programmatically.
- [ ] Text contrast >= 4.5:1; UI/focus contrast >= 3:1; no color-only meaning.
- [ ] Images have appropriate alt; media has captions/transcripts.
- [ ] ARIA is valid, minimal, and kept in sync with state.
- [ ] Primary flow verified with a real screen reader.
