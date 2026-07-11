# Privacy Masking

Configure masking selectors for sensitive customer fields so agents cannot view protected values.

Current 3.x changelog coverage extends masking beyond form inputs to `img`, `svg`, `iframe`,
canvas, text nodes, and background images. Verify each content type in both customer and agent
views; dynamic DOM updates and rendered canvas/image content need explicit regression tests.

See:
- [Features (official)](../references/features-official.md)
- [Get Started](../get-started.md)
