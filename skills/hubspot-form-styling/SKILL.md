---
name: hubspot-form-styling
description: HubSpot form CSS styling for embedded forms, CSS variables, form selectors, and modern form styling (NOT legacy forms). Use when styling HubSpot forms, customizing form appearance, or troubleshooting form CSS issues.
---

# HubSpot Form Styling (Modern Forms)

**IMPORTANT**: This guide covers the **updated form editor** styling. Legacy forms use different approaches and are being phased out.

## Key Differences: Modern vs Legacy Forms

| Feature | Modern Forms | Legacy Forms |
|---------|--------------|--------------|
| Raw HTML embed | NOT available | Available |
| CSS Variables | Supported | Not supported |
| Form Editor styling | Full control | Limited |
| Recommended approach | CSS variables + external CSS | Raw HTML + custom CSS |

## Styling Methods

### Method 1: CSS Variables (Recommended)

Override CSS variables globally or per-form:

```css
/* Global form styling */
:root {
  /* Typography */
  --hsf-global__font-family: 'Inter', sans-serif;
  --hsf-global__font-size: 16px;
  --hsf-global__color: #333333;

  /* Inputs */
  --hsf-input__background-color: #ffffff;
  --hsf-input__border-color: #cccccc;
  --hsf-input__border-radius: 4px;
  --hsf-input__padding: 12px 16px;

  /* Focus states */
  --hsf-input__focus-border-color: #0066cc;
  --hsf-input__focus-box-shadow: 0 0 0 3px rgba(0, 102, 204, 0.1);

  /* Labels */
  --hsf-label__font-size: 14px;
  --hsf-label__font-weight: 600;
  --hsf-label__color: #333333;

  /* Buttons */
  --hsf-button__background-color: #0066cc;
  --hsf-button__color: #ffffff;
  --hsf-button__border-radius: 4px;
  --hsf-button__padding: 12px 24px;
  --hsf-button__font-weight: 600;

  /* Button hover */
  --hsf-button__hover-background-color: #0052a3;

  /* Error states */
  --hsf-error__color: #dc3545;
  --hsf-error__border-color: #dc3545;

  /* Success */
  --hsf-success__background-color: #28a745;
}
```

### Method 2: External CSS Selectors

Common CSS selectors for form elements:

```css
/* Form container */
.hs-form {
  max-width: 600px;
}

/* Form fieldset (wraps each field) */
.hs-form-field {
  margin-bottom: 20px;
}

/* Labels */
.hs-form label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
}

/* Required indicator */
.hs-form-required {
  color: #dc3545;
}

/* All inputs */
.hs-form .hs-input {
  width: 100%;
  padding: 12px 16px;
  border: 1px solid #cccccc;
  border-radius: 4px;
  font-size: 16px;
  transition: border-color 0.2s, box-shadow 0.2s;
}

/* Input focus */
.hs-form .hs-input:focus {
  outline: none;
  border-color: #0066cc;
  box-shadow: 0 0 0 3px rgba(0, 102, 204, 0.1);
}

/* Text inputs specifically */
.hs-form input[type="text"],
.hs-form input[type="email"],
.hs-form input[type="tel"],
.hs-form input[type="number"] {
  height: 48px;
}

/* Textarea */
.hs-form textarea.hs-input {
  min-height: 120px;
  resize: vertical;
}

/* Select dropdowns */
.hs-form select.hs-input {
  height: 48px;
  background-image: url("data:image/svg+xml,...");
  background-repeat: no-repeat;
  background-position: right 12px center;
  padding-right: 40px;
  appearance: none;
}

/* Checkboxes & radios container */
.hs-form .inputs-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.hs-form .inputs-list li {
  margin-bottom: 8px;
}

/* Checkbox/radio inputs */
.hs-form .inputs-list input[type="checkbox"],
.hs-form .inputs-list input[type="radio"] {
  width: 18px;
  height: 18px;
  margin-right: 8px;
  vertical-align: middle;
}

/* Submit button */
.hs-form .hs-button {
  background-color: #0066cc;
  color: #ffffff;
  border: none;
  padding: 14px 28px;
  font-size: 16px;
  font-weight: 600;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.hs-form .hs-button:hover {
  background-color: #0052a3;
}

.hs-form .hs-button:focus {
  outline: none;
  box-shadow: 0 0 0 3px rgba(0, 102, 204, 0.3);
}

/* Error messages */
.hs-form .hs-error-msgs {
  list-style: none;
  padding: 0;
  margin: 4px 0 0;
}

.hs-form .hs-error-msgs li {
  color: #dc3545;
  font-size: 14px;
}

/* Error state on inputs */
.hs-form .hs-input.error,
.hs-form .error .hs-input {
  border-color: #dc3545;
}

/* Help text */
.hs-form .hs-field-desc {
  font-size: 14px;
  color: #666666;
  margin-top: 4px;
}

/* Placeholder text */
.hs-form .hs-input::placeholder {
  color: #999999;
}

/* Rich text in forms */
.hs-form .hs-richtext {
  margin-bottom: 20px;
}

/* Legal consent */
.hs-form .legal-consent-container {
  margin-top: 20px;
  font-size: 14px;
}

/* GDPR checkboxes */
.hs-form .hs-form-booleancheckbox-display {
  display: flex;
  align-items: flex-start;
  gap: 8px;
}

/* Multi-column forms */
.hs-form fieldset.form-columns-2 {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.hs-form fieldset.form-columns-3 {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 20px;
}

/* Responsive */
@media (max-width: 600px) {
  .hs-form fieldset.form-columns-2,
  .hs-form fieldset.form-columns-3 {
    grid-template-columns: 1fr;
  }
}
```

### Method 3: Disable Default Styling

In embed code, add `css: ''` to remove HubSpot defaults:

```html
<script>
  hbspt.forms.create({
    region: "na1",
    portalId: "YOUR_PORTAL_ID",
    formId: "YOUR_FORM_ID",
    css: ""  // Removes default HubSpot styling
  });
</script>
```

**Note**: This only removes default styling. Some structural CSS (like max-width) cannot be removed.

## Styling in Modules

When embedding forms in custom modules:

```css
/* module.css */

/* Wrapper for specificity */
.my-form-module .hs-form {
  /* Form styles */
}

.my-form-module .hs-form .hs-input {
  /* Input styles */
}

.my-form-module .hs-form .hs-button {
  /* Button styles */
}
```

```hubl
{# module.html #}
<div class="my-form-module">
  {% form
    form_to_use="{{ module.form.form_id }}"
  %}
</div>
```

## Theme-Consistent Forms

Use theme variables in form CSS:

```hubl
{# In template or module #}
<style>
  .hs-form .hs-button {
    background-color: {{ theme.colors.primary_color.color }};
    font-family: {{ theme.typography.primary_font.font }};
  }

  .hs-form .hs-input:focus {
    border-color: {{ theme.colors.primary_color.color }};
  }
</style>
```

## Common Form Classes Reference

| Class | Element |
|-------|---------|
| `.hs-form` | Form container |
| `.hs-form-field` | Field wrapper |
| `.hs-input` | Input elements |
| `.hs-button` | Submit button |
| `.hs-error-msgs` | Error message list |
| `.hs-field-desc` | Help text |
| `.inputs-list` | Checkbox/radio list |
| `.hs-richtext` | Rich text blocks |
| `.legal-consent-container` | Legal/GDPR section |
| `.submitted-message` | Success message |
| `.form-columns-2` | Two-column layout |
| `.form-columns-3` | Three-column layout |

## JavaScript-Based Styling

For dynamic styling after form loads:

```javascript
window.addEventListener('message', function(event) {
  if (event.data.type === 'hsFormCallback' && event.data.eventName === 'onFormReady') {
    const form = document.querySelector('.hs-form');

    // Add custom classes
    form.classList.add('my-custom-form');

    // Modify elements
    const inputs = form.querySelectorAll('.hs-input');
    inputs.forEach(input => {
      input.classList.add('my-input-class');
    });
  }
});
```

## Form in React Themes

For React themes, wrap the HubL form:

```hubl
{# In HubL template #}
<div id="contact-form-wrapper">
  {% form form_to_use="FORM_ID" %}
</div>
```

Style with scoped CSS:

```css
#contact-form-wrapper .hs-form {
  /* Scoped styles */
}
```

## Troubleshooting

### Styles Not Applying

1. **Specificity issues**: Add more specific selectors or use `!important`
2. **Load order**: Ensure your CSS loads after HubSpot's form CSS
3. **Embed timing**: Use JavaScript to apply styles after form loads

### Removing HubSpot Branding

Some structural styles cannot be removed. Work with them:

```css
/* Override max-width */
.hs-form {
  max-width: none !important;
}

/* Override other structural styles */
.hs-form fieldset {
  max-width: none !important;
}
```

### Form Not Matching Theme

Always use CSS variables or theme values:

```css
:root {
  --hsf-button__background-color: var(--theme-primary-color);
}
```

## Best Practices

1. **Use CSS Variables**: Most maintainable approach
2. **Scope Styles**: Wrap forms in a container for specificity
3. **Test Responsively**: Multi-column forms need mobile styles
4. **Accessibility**: Maintain focus states and contrast
5. **Error States**: Style error messages clearly
6. **Success State**: Style the `.submitted-message` container

## Overriding Default HubSpot Module Styles

HubSpot's default modules have aggressive CSS that often needs overriding.

### Accordion Module

```css
/* Override padding/sizing - requires !important */
.accordion-module .card-header {
  padding: 12px 16px !important;
}

.accordion-module .card-body {
  padding: 16px !important;
}
```

Pseudo-elements (chevron icons) cannot always be restyled with CSS alone. For stubborn pseudo-elements, inject styles via JavaScript after DOM load:

```javascript
document.addEventListener('DOMContentLoaded', function() {
  document.querySelectorAll('.accordion-trigger::after').forEach(el => {
    el.style.setProperty('background', 'your-gradient', 'important');
  });
});
```

### Headline Generator Module

```css
/* Override default container padding */
.headline-module .text-field-container {
  padding: 0 !important;
}
```

### Blocks Rich Text Design Module

```css
/* Override large default padding */
.hs_cos_wrapper_type_rich_text {
  padding: 0 !important;
}
```

### General Override Strategy

1. Inspect in browser DevTools to find the exact HubSpot class
2. Use a more specific selector (parent class + HubSpot class) for specificity
3. Use `!important` as a last resort when HubSpot's styles are too aggressive
4. For pseudo-elements that resist CSS, use JS injection on `DOMContentLoaded`

## Official Documentation

- [Forms Reference](https://developers.hubspot.com/docs/cms/building-blocks/forms)
- [Set up and style forms](https://knowledge.hubspot.com/forms/set-up-and-style-your-form-on-an-external-site)
