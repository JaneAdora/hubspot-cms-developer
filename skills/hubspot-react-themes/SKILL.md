---
name: hubspot-react-themes
description: HubSpot CMS React themes using js_partial, JavaScript partials, TypeScript support, webpack configuration, and hybrid React+HubL development. Use when building React-based themes, implementing interactive components, or working with the modern HubSpot development approach.
---

# HubSpot React + HubL Themes

React themes combine server-rendered HubL with client-side React interactivity.

## Key Differences from Traditional Themes

| Aspect | Traditional Theme | React Theme |
|--------|-------------------|-------------|
| **Development** | Design Manager or local CLI | Local development only |
| **Visibility** | Visible in Design Manager | NOT visible in Design Manager |
| **Editing** | Editable in Design Manager | Must deploy via CLI |
| **Technology** | HubL + HTML + CSS + vanilla JS | React + TypeScript + HubL |
| **Build** | None required | Webpack/Vite build process |

**IMPORTANT**: React themes are developed locally only. They are NOT visible or editable in the Design Manager. You must use the HubSpot CLI for all changes.

## Project Structure

```
my-react-project/
├── hubspot.config.yml          # HubSpot configuration
├── package.json                # Node dependencies
├── webpack.config.js           # Build configuration
├── src/
│   ├── components/
│   │   └── partials/           # React partials (required location)
│   │       ├── Header.jsx
│   │       ├── Footer.jsx
│   │       └── InteractiveWidget.tsx
│   ├── theme/
│   │   ├── theme.json
│   │   ├── fields.json
│   │   ├── templates/
│   │   │   └── home.html
│   │   ├── modules/
│   │   └── css/
│   └── lib/                    # Shared utilities
├── .hsignore
└── .gitignore
```

## Getting Started

### Create New React Project

```bash
npx @hubspot/create-cms-theme@latest
```

### hubspot.config.yml

```yaml
defaultPortal: YOUR_ACCOUNT_ID
portals:
  - name: production
    portalId: YOUR_PROD_ID
  - name: sandbox
    portalId: YOUR_SANDBOX_ID
```

### package.json

```json
{
  "name": "my-react-theme",
  "scripts": {
    "dev": "hs project dev",
    "build": "webpack --mode production",
    "deploy": "hs project upload"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@hubspot/cli": "^8.0.0",
    "@hubspot/webpack-cms-plugins": "^5.0.0",
    "@babel/core": "^7.23.0",
    "@babel/preset-react": "^7.23.0",
    "babel-loader": "^9.1.0",
    "webpack": "^5.89.0",
    "webpack-cli": "^5.1.0"
  }
}
```

## js_partial Tag

Include React components in HubL templates:

```hubl
{% js_partial
  path="@projects/my-project/components/partials/Header.jsx",
  pageTitle=content.name,
  navItems=menu("main-navigation").children
%}
```

### Syntax

```hubl
{% js_partial
  path="@projects/PROJECT_NAME/RELATIVE_PATH"
  prop1=value1
  prop2=value2
%}
```

- `path`: Required. Path to React component.
- Additional parameters become React props.

### Passing Data from HubL to React

```hubl
{# Template: templates/home.html #}
{% set blog_posts = blog_recent_posts(blog.id, 5) %}

{% js_partial
  path="@projects/my-theme/components/partials/BlogList.jsx",
  posts=blog_posts,
  showImages=true,
  maxItems=5
%}
```

```jsx
// components/partials/BlogList.jsx
export default function BlogList({ posts, showImages, maxItems }) {
  return (
    <ul className="blog-list">
      {posts.slice(0, maxItems).map(post => (
        <li key={post.id}>
          {showImages && post.featured_image && (
            <img src={post.featured_image} alt={post.name} />
          )}
          <a href={post.absolute_url}>{post.name}</a>
        </li>
      ))}
    </ul>
  );
}
```

## React Components

### Basic Component

```jsx
// components/partials/Hero.jsx
import React from 'react';

export default function Hero({ title, subtitle, ctaText, ctaUrl }) {
  return (
    <section className="hero">
      <h1>{title}</h1>
      {subtitle && <p>{subtitle}</p>}
      {ctaUrl && (
        <a href={ctaUrl} className="btn btn-primary">
          {ctaText || 'Learn More'}
        </a>
      )}
    </section>
  );
}
```

### TypeScript Component

```tsx
// components/partials/Card.tsx
import React from 'react';

interface CardProps {
  title: string;
  description?: string;
  imageUrl?: string;
  link?: {
    url: string;
    openInNewTab: boolean;
  };
}

export default function Card({
  title,
  description,
  imageUrl,
  link
}: CardProps) {
  return (
    <article className="card">
      {imageUrl && <img src={imageUrl} alt="" className="card__image" />}
      <div className="card__content">
        <h3 className="card__title">{title}</h3>
        {description && <p>{description}</p>}
        {link?.url && (
          <a
            href={link.url}
            target={link.openInNewTab ? '_blank' : undefined}
            rel={link.openInNewTab ? 'noopener noreferrer' : undefined}
          >
            Read More
          </a>
        )}
      </div>
    </article>
  );
}
```

### Interactive Component

```tsx
// components/partials/Accordion.tsx
import React, { useState } from 'react';

interface AccordionItem {
  title: string;
  content: string;
}

interface AccordionProps {
  items: AccordionItem[];
  allowMultiple?: boolean;
}

export default function Accordion({
  items,
  allowMultiple = false
}: AccordionProps) {
  const [openIndices, setOpenIndices] = useState<number[]>([]);

  const toggleItem = (index: number) => {
    if (allowMultiple) {
      setOpenIndices(prev =>
        prev.includes(index)
          ? prev.filter(i => i !== index)
          : [...prev, index]
      );
    } else {
      setOpenIndices(prev =>
        prev.includes(index) ? [] : [index]
      );
    }
  };

  return (
    <div className="accordion">
      {items.map((item, index) => (
        <div key={index} className="accordion__item">
          <button
            className="accordion__trigger"
            onClick={() => toggleItem(index)}
            aria-expanded={openIndices.includes(index)}
          >
            {item.title}
          </button>
          {openIndices.includes(index) && (
            <div className="accordion__content">
              {/*
                Note: For rich text from HubSpot, content is already sanitized.
                For user-generated content, use DOMPurify before rendering.
              */}
              <div dangerouslySetInnerHTML={{ __html: item.content }} />
            </div>
          )}
        </div>
      ))}
    </div>
  );
}
```

**Security Note**: When rendering HTML content in React, HubSpot's rich text fields are pre-sanitized. For any user-generated content, always sanitize with a library like DOMPurify before rendering.

## Webpack Configuration

```javascript
// webpack.config.js
const path = require('path');
const HubSpotAutoUploadPlugin = require('@hubspot/webpack-cms-plugins/HubSpotAutoUploadPlugin');

module.exports = ({ account, autoupload }) => ({
  mode: 'development',
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              '@babel/preset-env',
              '@babel/preset-react',
              '@babel/preset-typescript'
            ]
          }
        }
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  },
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx']
  },
  plugins: [
    new HubSpotAutoUploadPlugin({
      autoupload,
      account,
      src: 'dist',
      dest: 'my-theme'
    })
  ]
});
```

## Development Workflow

### Local Development

```bash
# Start development with auto-upload
hs project dev

# Or using npm scripts
npm run dev
```

### Deployment

**React themes deployment is different from traditional themes:**

```bash
# Build and deploy
npm run build
hs project upload

# Or deploy specific folder
hs upload src/theme my-theme --account=production --use-env
```

### Preview

`hs theme preview` was removed in CLI v8.0.0. Use the upload-and-preview workflow instead:

```bash
# Upload to sandbox and preview on HubSpot
hs watch src/theme my-theme --initial-upload --account=sandbox
```

### Serverless Functions

Serverless functions are **NOT available** in v2025.2 projects. They return in v2026.03 (March 2026). Until then, host functions externally.

## Styling React Components

### CSS Modules

```tsx
// components/partials/Button.tsx
import React from 'react';
import styles from './Button.module.css';

export default function Button({ children, variant = 'primary' }) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      {children}
    </button>
  );
}
```

### Inline Styles with Theme Values

Pass theme values from HubL:

```hubl
{% js_partial
  path="@projects/my-theme/components/partials/Section.jsx",
  backgroundColor=theme.colors.primary_color.color,
  textColor=theme.colors.text_color.color
%}
```

```jsx
export default function Section({ children, backgroundColor, textColor }) {
  return (
    <section style={{ backgroundColor, color: textColor }}>
      {children}
    </section>
  );
}
```

## Using HubL Data in React

### Blog Posts

```hubl
{% set posts = blog_recent_posts(blog.id, 10) %}
{% set serialized_posts = [] %}
{% for post in posts %}
  {% do serialized_posts.append({
    "id": post.id,
    "title": post.name,
    "url": post.absolute_url,
    "image": post.featured_image,
    "excerpt": post.post_summary|truncate(150),
    "date": post.publish_date|datetimeformat("%Y-%m-%d")
  }) %}
{% endfor %}

{% js_partial
  path="@projects/my-theme/components/partials/BlogGrid.jsx",
  posts=serialized_posts
%}
```

### HubDB Data

```hubl
{% set rows = hubdb_table_rows(TABLE_ID) %}
{% js_partial
  path="@projects/my-theme/components/partials/DataTable.jsx",
  data=rows
%}
```

### CRM Data

```hubl
{% set contact = request_contact() %}
{% if contact %}
  {% js_partial
    path="@projects/my-theme/components/partials/PersonalizedContent.jsx",
    firstName=contact.firstname,
    company=contact.company
  %}
{% endif %}
```

## Best Practices

1. **SEO-Critical Content in HubL**: Render important content server-side with HubL, enhance with React
2. **Progressive Enhancement**: Site should work without JavaScript
3. **Hydration**: Be mindful of hydration mismatches between server and client
4. **Bundle Size**: Code-split large components
5. **Accessibility**: Ensure interactive components are keyboard accessible
6. **Testing**: Use Storybook to develop components in isolation

## Common Issues

### Component Not Rendering
- Check path in `js_partial` is correct
- Verify component is in `components/partials/` directory
- Ensure component has a default export

### Hydration Mismatch
- Server-rendered HubL content must match initial React render
- Avoid using `Date.now()` or random values during initial render

### Styles Not Loading
- Ensure CSS is imported in the component
- Check webpack is processing CSS files
