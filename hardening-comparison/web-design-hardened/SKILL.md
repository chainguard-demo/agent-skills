---
name: web-design-guidelines
description: Review UI code for Web Interface Guidelines compliance. Use when asked to "review my UI", "check accessibility", "audit design", "review UX", or "check my site against best practices".
metadata:
  author: vercel
  version: "1.0.0"
  argument-hint: <file-or-pattern>
allowed-tools:
  - read-file
---

# Web Interface Guidelines

Review files for compliance with Web Interface Guidelines.

## How It Works

1. Read the specified files (or prompt user for files/pattern).
2. Check the code against all rules in the Guidelines section below.
3. Output findings in the specified format.

## Guidelines

The following rules are checked during the review.

### Accessibility (a11y)

- **`img-alt`**: All `<img>` elements must have a non-empty `alt` attribute.
- **`label-for`**: All form `<input>`, `<textarea>`, and `<select>` elements must have an associated `<label>`.
- **`button-name`**: All `<button>` elements must have discernible text content or an `aria-label`.
- **`color-contrast`**: Text and background colors must have a contrast ratio of at least 4.5:1 for normal text and 3:1 for large text.

### Usability

- **`target-size`**: Interactive elements like buttons and links should have a minimum target size of 44x44 CSS pixels.
- **`no-autoplay`**: `<video>` and `<audio>` elements should not autoplay unless muted and without user interaction.

### Best Practices

- **`no-important`**: Avoid using `!important` in CSS rules as it breaks the cascade.
- **`secure-links`**: Links that open in a new tab (`target="_blank"`) must include `rel="noopener noreferrer"` to prevent security vulnerabilities.

## Output Format

Findings are reported in a terse `file:line:column` format:

`path/to/file.js:line:column: rule-id: Brief description of the issue.`

### Example

```
src/components/Avatar.jsx:12:5: img-alt: <img> element is missing an 'alt' attribute.
src/styles/forms.css:45:10: no-important: Avoid using '!important'.
```

## Usage

When a user provides a file or pattern argument:
1. Read the specified files.
2. Apply all rules from the `Guidelines` section.
3. Output findings using the format specified above.

If no files are specified, ask the user which files to review.
