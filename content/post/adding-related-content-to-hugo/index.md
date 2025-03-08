---
title: "How to Add Related Content to Your Hugo Theme"
date: 2023-10-15
description: "A comprehensive guide on implementing and customizing related content in Hugo websites"
tags: ["hugo", "web development", "tutorial", "related content"]
categories: ["web development", "tutorials"]
image: cover.png
---

## Why Related Content Matters

Related content sections are a powerful way to keep visitors engaged with your website by suggesting relevant articles they might be interested in. They reduce bounce rates, increase page views, and improve the overall user experience by helping readers discover more of your content.

## How Hugo Handles Related Content

Hugo has a built-in system for identifying and displaying related content based on various factors like tags, categories, and publication dates. Unlike some other platforms that might use complex algorithms or third-party services, Hugo handles this functionality natively.

## Configuring Related Content in Hugo

### Step 1: Add Related Content Configuration

The first step is to add the related content configuration to your `hugo.toml` (or `config.yaml`/`config.json`):

```toml
[related]
    # Include newer content in the relatedness calculation
    includeNewer = true
    
    # The minimum score a page must have to be considered related
    threshold = 60
    
    # Convert tags/categories to lowercase before comparison
    toLower = false
    
    [[related.indices]]
        name = "tags"
        weight = 100
    
    [[related.indices]]
        name = "categories"
        weight = 80
    
    [[related.indices]]
        name = "date"
        weight = 10
```

This configuration tells Hugo:
- To include newer content in the related content calculation (`includeNewer = true`)
- That a page must score at least 60 points to be considered related (`threshold = 60`)
- To match tags with a weight of 100, categories with a weight of 80, and consider publication date with a weight of 10

### Step 2: Make Sure Your Content is Well-Tagged

For Hugo to find related content, your posts need to be properly tagged and categorized. Add tags and categories to the front matter of your Markdown posts:

```yaml
---
title: "My Amazing Post"
date: 2023-10-01
tags: ["hugo", "web development", "jamstack"]
categories: ["tutorials", "web"]
---
```

The more specific and consistent your tagging system is, the better Hugo can match related content.

### Step 3: Styling the Related Content Section

If you're using the Hugo Stack theme like I am, the theme already includes basic styling for the related content section. However, you can customize it further by adding custom CSS:

```css
/* Custom styles for related content */
.article-list--compact.related-content {
    margin-top: 2rem;
    border-top: 1px solid var(--card-separator-color);
    padding-top: 1.5rem;
}

.article-list--compact.related-content .article-title {
    font-size: 1.1rem;
    font-weight: 600;
}

.related-content header {
    margin-bottom: 1rem;
}

.related-content header h2 {
    font-size: 1.4rem;
    color: var(--accent-color);
}
```

Add this CSS to your site's custom CSS file (usually in `static/css/custom.css`) and make sure it's included in your site configuration.

## Best Practices for Effective Related Content

### 1. Use Specific and Consistent Tagging

Tags are the primary way Hugo identifies related content. Use specific, descriptive tags and be consistent with your tagging system.

### 2. Organize Content with Categories

Categories provide another dimension for relating content. While tags can be more specific and numerous, categories should represent broader classifications of your content.

### 3. Review and Adjust Related Content Settings

Monitor how your related content appears on your site. If you find that Hugo is suggesting unrelated content, you might need to:
- Increase the threshold value
- Adjust the weights of different indices
- Improve your tagging strategy

### 4. Consider Manual Related Content Links

For especially important connections that Hugo might miss, you can manually add links to related content within your posts:

```markdown
## Related Articles
- [Getting Started with Hugo](/posts/getting-started-with-hugo/)
- [Advanced Hugo Templates](/posts/advanced-hugo-templates/)
```

## Example: How Related Content Calculation Works

Let's say we have three posts:

**Post A**: Tags: [hugo, markdown, templates], Categories: [tutorials]
**Post B**: Tags: [hugo, css, design], Categories: [tutorials]
**Post C**: Tags: [wordpress, cms], Categories: [reviews]

When displaying related content for Post A:
- Post B scores: 100 (shared tag "hugo") + 80 (shared category "tutorials") = 180
- Post C scores: 0 (no shared tags or categories)

Since Post B exceeds our threshold of 60, it will be shown as related content, while Post C will not.

## Conclusion

Implementing related content in your Hugo site is a straightforward but powerful way to enhance the user experience and keep visitors engaged with your content. By properly configuring Hugo's related content settings and maintaining a consistent tagging system, you can automatically suggest relevant articles to your readers and increase the time they spend on your site.

By using the built-in functionality of Hugo and the Hugo Stack theme, you can have an elegant related content section without any third-party services or complex plugins.
