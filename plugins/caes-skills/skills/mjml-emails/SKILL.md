---
name: mjml-emails
description: Use when creating, migrating, replacing, wiring, styling, or testing MJML-based email templates, shared email layouts, MJML partials/components, email view models, render services, send/queue services, or MJML render tests.
---

# MJML Emails

## Workflow

1. Find the existing email system before editing:
   - Template folders
   - Shared layout
   - Shared partials/components
   - Model/view-model folder
   - Renderer service
   - Send/queue service
   - Render tests or snapshot tests
2. If replacing a legacy email, compare against the current template first. Preserve subject, recipients, body content, bypass/admin behavior, links, totals, and conditional sections unless the user asks for a behavior change.
3. Add or update templates following local conventions:
   - Same folder structure
   - Same naming pattern
   - Same layout
   - Same model style
   - Same render/service tests
4. Reuse shared MJML partials/components before writing new markup.
5. Add a new component when content is reusable, conditional, or independently testable.
6. Wire service calls at the same abstraction level as existing emails. Avoid bypassing queue/render services unless the project already does.

## MJML Reference

Use the official MJML documentation as the primary reference for MJML syntax, supported components, attributes, and rendering behavior:

https://documentation.mjml.io/

Prefer project-local email patterns over generic examples from the documentation when they conflict, but make sure to clearly inform the user of these conflicts and explain the reasoning.

## Styling

- Prefer shared `mj-class` definitions in the common layout for repeated MJML attributes:
  - Headings
  - Body copy
  - Table wrappers
  - Buttons
  - Footer text
  - Section/container spacing
- Use `mj-class` on MJML elements such as `mj-text`, `mj-table`, `mj-button`, `mj-section`, and `mj-column`.
- Keep critical raw HTML table-cell styles inline inside `mj-table`; email clients handle inline `td` styles more reliably than CSS classes.
- Do not introduce broad visual redesigns during a behavior/template migration. Match existing spacing, colors, typography, and card treatment unless asked.

## Links And Files

- Prefer secure application routes in emails over direct storage/blob URLs.
- If a file is downloadable, link to an app route that authorizes, logs, and redirects to the actual file.
- Do not attach files unless the queue model, sender, retry flow, and storage access explicitly support attachments.
- Feature-flag optional or risky email content when rollout control matters.

## Conditional Content

- Put reusable conditional blocks in separate partials/components.
- Keep the parent template responsible for deciding whether the component renders.
- Keep the component defensive: if required URL, text, or model data is missing, render nothing.

## Testing

Add or update tests that cover:

- Service method chooses the correct template
- Subject and recipients
- Bypass/admin behavior
- Feature-flag on/off behavior
- Rendered HTML contains important user-visible text
- Rendered HTML does not contain MJML tags
- Omitted conditional content is absent when expected

For MJML render tests, assert durable content, not exact generated HTML structure.

## Review Checklist

Before finishing:

- Does the template use the project's shared layout?
- Are repeated styles using existing `mj-class` values?
- Are raw `td` styles still inline where email-client compatibility matters?
- Are links absolute when emails require external navigation?
- Are optional sections behind data checks or feature flags?
- Are subject, recipients, and bypass behavior preserved?
- Did render tests exercise the template?
